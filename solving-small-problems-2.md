---
id: ghost-27
title: Solving Small Problems - Blogging Platform #4.2
author: Grzegorz Ziemonski
summary: *This post is a continuation of the previous one. [Click here](http://tidyjava.com/solving-small-problems) to read it, if you haven't done so yet.*
date: 2016-08-22
tags:
    - java
    - blogging-platform
---
### Reading Cloned Files

This should be pretty straightforward. It should be enough to use JDK `File` API and read from the `.contents`. Let's do this.

![](/img/read.png)

*&#9834;&#9834;&#9834; sounds of coding &#9834;&#9834;&#9834;*

STOP, STOP, STOP!

Didn't you see that? This tiny little `PostReader` knows too much. The concept `.contents` directory is specific to the way I'm implementing Git support. I believe we now got a... `GitPostReader`!

*&#9834;&#9834;&#9834; sounds of coding &#9834;&#9834;&#9834;*

I merged `PostReader` with `GitCloner` to achieve a cohesive whole:

```java
@Service
public class GitPostReader {

    /* cloning logic as before */

    public List<Post> readAll() {
        File contentsDir = gitRepository.getWorkTree();
        return Stream.of(contentsDir
                .listFiles(withSupportedExtension()))
                .map(MarkdownPostFactory::create)
                .collect(Collectors.toList());
    }

    private FilenameFilter withSupportedExtension() {
        return (dir, name) -> name.endsWith(MarkdownPostFactory.EXTENSION);
    }

    public Post readOne(String name) {
        File postFile = new File(toPostPath(name));
        if (!postFile.exists()) {
            throw new MissingPostException();
        }
        return MarkdownPostFactory.create(postFile);
    }

    private String toPostPath(String name) {
        return gitRepository.getWorkTree().getPath() + name + MarkdownPostFactory.EXTENSION;
    }
}
```

The fact that I merged these 2 classes is not that important. Actually, if there was more logic in each of them, they would probably stay separate. The important point is that `PostReader` became coupled to Git support, which effectively made it a `GitPostReader`.

### Commit Hook

![](/img/hook.jpg)

Adding a commit hook sounds easy. It should be enough to create an endpoint to call a `git pull` command.

*&#9834;&#9834;&#9834; sounds of coding &#9834;&#9834;&#9834;*

As expected, this one was easy (to write).

```java
@RestController
public class GitController {

    @Autowired
    private GitPostReader gitPostReader;

    @RequestMapping(path = "/git/commit", method = RequestMethod.POST)
    public void postCommit() {
        gitPostReader.pullChanges();
    }
}
```

In `GitPostReader`, I created a reference to `Git` object and added `pullChanges` method:

```java
@Service
public class GitPostReader {

    /* nothing new */

    private Git git;

    /* nothing new */

    public void pullChanges() {
        git.pull();
    }
}
```

I have no idea how to test it without leveraging more of JGit APIs. I'm not sure if it's required, since I'm only delegating to JGit. For now, I created an integration test:

```groovy
@SpringApplicationConfiguration(BloggingPlatform.class)
@WebAppConfiguration
class GitControllerIntegrationSpec extends Specification {

    @Autowired
    WebApplicationContext wac

    MockMvc mockMvc

    def setup() {
        mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build()
    }

    def "post commit hook"() {
        expect:
        mockMvc.perform(post("/git/commit"))
                .andExpect(status().isOk())
    }
}
```

### Testing

Final step, I have to test the whole thing. To be honest, while developing I used the old test. I configured the application to use the [hello world repository](https://github.com/tidyjava/blogging-platform-hello-world). Then, I ran it and checked if it fails with a single README post with TILT's and a header.

As before, I'd like to test most cases via integration tests. It makes sense to me that integration tests would use a real, online Git repostory. In this case, it should be enough to move old test posts into an online repo and point my tests to use it. I'll also add a "TILT post" to check that the application doesn't crash in case of missing metadata fields.

*&#9834;&#9834;&#9834; sounds of coding &#9834;&#9834;&#9834;*

As said above, I moved test posts to a special repository. You can see it [here](https://github.com/tidyjava/blogging-platform-test-contents).

I modified `home` test and added special `tilt post` test. For the "TILT post", I created a special `assertTiltPost` method.

```groovy
@SpringApplicationConfiguration(BloggingPlatform.class)
@WebAppConfiguration
@DirtiesContext
class PostControllerIntegrationSpec extends Specification {

    /* nothing new */

    def "home"() {
        expect:
        def result = mockMvc.perform(get("/"))
                .andExpect(status().isOk())
                .andExpect(view().name("home"))
                .andReturn()
        def posts = result.modelAndView.model.posts
        assertTestPost(posts[0], 2)
        assertTestPost(posts[1], 1)
        assertTiltPost(posts[2])
    }

    /* nothing new */

    def "tilt post"() {
        expect:
        def result = mockMvc.perform(get("/tilt"))
                .andExpect(status().isOk())
                .andExpect(view().name("post"))
                .andReturn()
        assertTiltPost(result.modelAndView.model.post)
    }

    /* nothing new */

    void assertTiltPost(post) {
        assert post.title == "TILT"
        assert post.summary == "TILT"
        assert post.date.format(ISO_LOCAL_DATE) == "1970-01-01"
        assert post.url == "/tilt"
        assert post.content == ""
    }
}
```

When preparing the tests, I found 2 bugs:

* `Git` object leaks file handles if not closed
* I forgot to sort posts when reading all of them for the home page

Well, the test suite has proven itself trustworthy!

### That's it!

![](/img/final.jpg)

Oh boy/girl, that was fun! It was also easy, despite the fact that I used a new library and I didn't know how to test the feature at start. Step by step, small problem by small problem, I got the most important feature running in a few hours. If you experienced problems with mental overload during analysis before, you now know what to do - **solve small problems!**

Before I merge the changes, the application will be available [here](https://blogging-platform-part4.herokuapp.com/). Afterwards, it will be available under [standard link](https://blogging-platform.herokuapp.com/). If you'd like to read final version of the code, contribute or w/e, project's repository is [here](https://github.com/tidyjava/blogging-platform).