---
id: ghost-31
title: Initial Implementation - Blogging Platform #3
author: Grzegorz Ziemonski
summary: Here it is, third part of the Blogging Platform series. If you don't know what I'm talking about, make sure to check out [previous parts](http://tidyjava.com/tag/blogging-platform/). For this part I finally wrote some meaningful code, so let's jump straight into it.
date: 2016-07-29
tags:
    - java
    - blogging-platform
---
### Tests
I started by writing some tests, but inspired by [James Coplien](http://rbcs-us.com/documents/Why-Most-Unit-Testing-is-Waste.pdf) and [Simon Brown](https://www.youtube.com/watch?v=SV5RVzKZueA) I decided to give up on TDD and heavy unit testing in this project.[^1] Instead, I wrote two coarser grain tests to excersise my web endpoints.

```groovy
    def "home"() {
        expect:
        def result = mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andExpect(view().name("home"))
                .andReturn()
        def posts = result.modelAndView.model.posts
        assertTestPost(posts[0], 2)
        assertTestPost(posts[1], 1)
    }

    def "post"(n) {
        expect:
        def result = mockMvc.perform(get("/post$n"))
                .andExpect(status().isOk())
                .andExpect(view().name("post"))
                .andReturn()
        assertTestPost(result.modelAndView.model.post, n)

        where:
        n << [1, 2]
    }
```

As you can see, I used MockMvc to expect a view and perform assertions on the model. There's a good reason for this. If someone forks the project to use it for his own blog, he should be able to modify view templates any way he wants, without breaking the tests. If I made any assertions on the content, I wouldn't be able to guarantee that.

During the development, one more test case emerged: missing post.

```groovy
    def "missing post"() {
        expect:
        mockMvc.perform(get("/surely-not-existent"))
                .andExpect(status().isNotFound())
                .andExpect(view().name("not-found"))
    }
```

Believe it or not, but these 3 tests, give me a very reasonable coverage. Actually it's 100%, if we don't count handling IO checked exceptions, which I want to rethrow anyway. The secret lies in the tests posts and `assertTestPost()` method.

```markdown
---
title: Post 1
summary: Summary 1
date: 1970-01-01
---

**Content 1**
```

```groovy
void assertTestPost(post, n) {
    assert post.title == "Post $n"
    assert post.summary == "Summary $n"
    assert post.date.format(ISO_LOCAL_DATE) == "1970-01-0$n"
    assert post.url == "/post$n"
    assert post.content == "<p><strong>Content $n</strong></p>\n"
}
```

To pass these assertions, the application has to correctly parse posts' markdown. Yet the example is simple enough to easily see what's happening. I didn't put any sophisticated markdown content in the posts, because my purpose is not to test [commonmark](https://github.com/atlassian/commonmark-java), it's to test that *some* parsing happened.

### Controller

The controller has grown from a single method to a stunning number of 4, one of which is a model attribute.

```java
@Controller
public class PostController {

    @Value("${blog.name}")
    private String blogName;

    @Autowired
    private PostReader postReader;

    @ModelAttribute("blogName")
    public String getBlogName() {
        return blogName;
    }

    @RequestMapping("/")
    public String home(Model model) {
        model.addAttribute("posts", postReader.readAll());
        return "home";
    }

    @RequestMapping("/{path}")
    public String post(@PathVariable("path") String path, Model model) {
        model.addAttribute("post", postReader.readOne(path));
        return "post";
    }

    @ResponseStatus(HttpStatus.NOT_FOUND)
    @ExceptionHandler(PostReader.MissingPost.class)
    public String missingPost() {
        return "not-found";
    }
}
```

Basically, we have 3 cases here: home page, post page or not-found page when a post is missing. Nothing special, all the magic seems to be somewhere else!

### PostReader

For reading the posts I created a `PostReader` class with 2 methods: `readAll()` and `readOne()`. Currently, it reads posts from files on the classpath. No Git and no caching so far, but I bet it will be needed (and done) soon.

```java
@Service
public class PostReader {

    @Value("${posts.location}")
    private String postsLocation;

    @Autowired
    private PathMatchingResourcePatternResolver resourceResolver;

    public List<MarkdownPost> readAll() {
        try {
            return Stream.of(resourceResolver.getResources(postLocation("*")))
                    .map(MarkdownPost::new)
                    .sorted(Comparator.comparing(MarkdownPost::getDate).reversed())
                    .collect(toList());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public MarkdownPost readOne(String path) {
        Resource resource = resourceResolver.getResource(postLocation(path));
        if (!resource.exists()) {
            throw new MissingPost();
        }
        return new MarkdownPost(resource);
    }

    private String postLocation(String path) {
        return postsLocation + path + MarkdownPost.EXTENSION;
    }

    public static class MissingPost extends RuntimeException {
    }
}
```

As you can see, it all comes down to creating a `MarkdownPost` from found resources. I used ugly-named `PathMatchingResourcePatternResolver`, because loading files from JAR resources turned out to be non-trivial and I didn't have too much time to spend on it.

### MarkdownPost

I wondered how to do this one. Obviously the post is supposed to hold the content and the metadata. I wasn't sure if it should be a stupid data holder or something smarter. I ended up with the second option, because it's simpler and seemed more natural - in the end, a post in the platform is a resource that we parse to get data. Here it is:

```java
public class MarkdownPost {
    public static final String EXTENSION = ".md";

    private Node parsedResource;
    private Map<String, List<String>> metadata;
    private String url;

    public MarkdownPost(Resource resource) {
        try {
            this.parsedResource = parse(resource);
            this.metadata = extractMetadata(parsedResource);
            this.url = "/" + resource.getFilename().replace(EXTENSION, "");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private Node parse(Resource resource) throws IOException {
        Parser parser = Parser.builder().extensions(singletonList(YamlFrontMatterExtension.create())).build();
        return parser.parseReader(new InputStreamReader(resource.getInputStream()));
    }

    private Map<String, List<String>> extractMetadata(Node document) {
        YamlFrontMatterVisitor visitor = new YamlFrontMatterVisitor();
        document.accept(visitor);
        return visitor.getData();
    }

    public String getTitle() {
        return metadata.get("title").get(0);
    }

    public String getSummary() {
        return metadata.get("summary").get(0);
    }

    public LocalDate getDate() {
        return LocalDate.parse(metadata.get("date").get(0), DateTimeFormatter.ISO_LOCAL_DATE);
    }

    public String getUrl() {
        return url;
    }

    public String getContent() {
        return HtmlRenderer.builder().build().render(parsedResource);
    }
}
```

I don't like the "smart constructor", but it hasn't caused any problems so far. Ideas how to improve it (or voices to leave it like this) are welcome.

### Views

I have created some simple views for demonstration purposes, but it's something I will have to spend a lot of time later to make the app look good.

### Wrap Up

That's it! Three tests and three classes: controller delegating most work to the reader, reader that transforms resources into markdown posts and the posts using a library to extract some data. So simple and yet it works like charm. You can see it working [here](http://blogging-platform.herokuapp.com/). All source code is avaliable [here](https://github.com/tidyjava/blogging-platform).

[^1]: It's not like I'm already converted and I'm against unit testing. I just wanted to try out something new.