---
id: ghost-26
title: Solving Small Problems - Blogging Platform #4.1
author: Grzegorz Ziemonski
summary: One of the keys to being a good programmer is working in an organized way. You pick a small, easy problem, solve it and move to the next small, easy one. One after another, step by step, something greater emerges. Our brains have limited capacity and processing speed. We can't do everything at once, we can't solve multiple complicated problems at the same time. Knowing this we can conciously limit our current "context" to something simple enough to chew at once.
date: 2016-08-22
tags:
    - java
    - blogging-platform
---
*Although I tried to make this post comprehensible by itself, you might want to read [previous parts](http://tidyjava.com/tag/blogging-platform/) before this one.*

One of the keys to being a good programmer is working in an organized way. You pick a small, easy problem, solve it and move to the next small, easy one. One after another, step by step, something greater emerges. Our brains have limited capacity and processing speed. We can't do everything at once, we can't solve multiple complicated problems at the same time. Knowing this we can conciously limit our current "context" to something simple enough to chew at once.

In this post, I'm going to show you how to put it in practice by developing a key feature of my [blogging platform](http://tidyjava.com/tag/blogging-platform/) like this. Get ready for a ride!

### Git Support
The key feature of the blogging platform is to load the posts from a separate Git repository, so that the bloggers can benefit from version control, code reviews, people's contributions and so on.

![](/img/start.png)

Currently, the posts are kept in the same repository as the platform and are loaded from the classpath using some ugly `PathMatchingResourcePatternResolver`. I want to replace the whole classpath-based solution with Git support. Ability to run the application in "local mode" by specifying local address of a repository would be a plus, but is not a must.

### Creating a Plan
One easy way to prevent your brain from exploding is to create a plan - set of simple steps to achieve your goal. The plan is not set in stone, it can evolve while developing and learning new things about the problem, but it needs to remain simple. Here's my initial plan for this feature:

1. Refactor `MarkdownPost`. Currently, it's coupled to the Spring's `Resource` class, has a nasty smart constructor and some other flaws.
2. Create a mechanism to clone a specified Git repository to the local filesystem.
3. Read cloned contents - make `PostReader` use cloned files instead of classpath resources.
4. Add an endpoint for a Git hook to refresh the files on commit.
5. Create automated tests for the whole thing.

Fifth point is currently very abstract and might be broken into smaller steps later. Currently, I don't have any good idea how to create or test Git-related mechanisms, so I can't tell what exactly will be done there.

### Refactoring MarkdownPost
The idea to refactor `MarkdownPost` came from 2 sources:

* comments under previous posts, both on my personal blog and DZone - that constructor and commonmark related logic hurt eyes
* my thoughts about the concept of a post in the project - it's not any high-level business entity. This platform is a Read application, which is even less than CRUD. I take a file, read whatever's relevant for a post and pass it further. Therefore, a post is a holder of whatever's relevant!

![](/img/refactor.png)

This leads to the following steps:

* rename `MarkdownPost` to `Post`
* create relevant fields in the post - `title`, `summary` etc.
* initialize them during creation of the post
* free the `Post` from dependency on Spring's `Resource`
* move parsing markdown from constructor to a factory method
* consider moving factory method to a standalone factory
* adjust `PostReader` accordingly

Yay, even simpler steps! Actually, most often you won't put so much details into your plan, but in this case I had a lot of input from readers and time to think before I found time to implement anything.

*&#9834;&#9834;&#9834; sounds of refactoring &#9834;&#9834;&#9834;*

Showing all intermediate steps would be even more boring than this, so I'll just show you the final effect.

`MarkdownPost` has turned into a simple data holder `Post`:

```java
public class Post {
    private String title;
    private String summary;
    private LocalDate date;
    private String url;
    private String content;

    public Post(String title, String summary, LocalDate date, String url, String content) {
        this.title = title;
        this.summary = summary;
        this.date = date;
        this.url = url;
        this.content = content;
    }

    /* getters */
}
```

Markdown related logic sits in a separate `MarkdownPostFactory`:

```java
public class MarkdownPostFactory {
    public static final String EXTENSION = ".md";

    public static Post create(InputStream inputStream, String filename) {
        try {
            Node parsedResource = parse(inputStream);
            Map<String, List<String>> metadata = extractMetadata(parsedResource);

            String title = getField(metadata, "title");
            String summary = getField(metadata, "summary");
            LocalDate date = toLocalDate(getField(metadata, "date"));
            String url = toUrl(filename);
            String content = toHtml(parsedResource);

            return new Post(title, summary, date, url, content);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    /* nothing new */

    private static String getField(Map<String, List<String>> metadata, String field) {
        return metadata.getOrDefault(field, asList("TILT")).get(0); // TODO test TILT
    }

    /* nothing new */
}
```

You can notice that I added a `TODO` to test a case when there's missing metadata in the post. I don't want to do it right now, because this class might change really soon when integrating with Git.

`PostReader` was adjusted to use `Post` and the factory. In the process, I also renamed `MissingPost` to `MissingPostException` and made it an outer, public class.

### Cloning a Git repository

![](/img/clone.png)

I've seen somewhere that there's a Java library called JGit, developed by some smart people from big companies. I'll add it to the project and try to clone a repo using it.

*&#9834;&#9834;&#9834; sounds of coding &#9834;&#9834;&#9834;*

It turned out to be suprisingly easy. 4 lines of a chained call and repository is cloned to our disk. When testing, I found out that we have to clear the repository directory manually, which I decided to brute force by deleting it and creating again.

```java
@Service
public class GitCloner {

    @Value("${blog.repositoryUrl}")
    private String repositoryUrl;

    @PostConstruct
    public void cloneRepository() {
        File contentsDir = new File(".contents");
        clean(contentsDir);
        cloneTo(contentsDir);
    }

    private void clean(File dir) {
        rethrow(() -> FileUtils.deleteDirectory(dir));
        dir.mkdirs();
    }

    private void cloneTo(File contentsDir) {
        rethrow(() -> Git.cloneRepository()
                .setURI(repositoryUrl)
                .setDirectory(contentsDir)
                .call());
    }
}
```

To aid development, I created a simple test for the class. It verifies that the contents directory is cleaned and repository cloned correctly.

```groovy
class GitClonerSpec extends Specification {
    static final CONTENTS_DIRECTORY = ".contents"
    static final CLONE_BLOCKER = Paths.get(CONTENTS_DIRECTORY + "/would-block-clone.txt");

    def gitCloner = new GitCloner()

    def setup() {
        gitCloner.repositoryUrl = "https://github.com/tidyjava/blogging-platform-hello-world.git"
    }

    def 'should clean and clone'() {
        given:
        Files.createFile(CLONE_BLOCKER)

        when:
        gitCloner.cloneRepository()

        then:
        Files.exists(Paths.get(CONTENTS_DIRECTORY + "/README.md"))
        !Files.exists(CLONE_BLOCKER)
    }
}
```

This test might change in the final step, when I'll decide for a testing strategy.

As you maybe saw, a `rethrow` construct appeared. I got pissed off by catching and rethrowing `IOException`s left and right, so I created `ExceptionUtils`:

```java
public class ExceptionUtils {

    public static <T> T rethrow(ThrowingSupplier<T> supplier) {
        try {
            return supplier.get();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void rethrow(ThrowingRunnable runnable) {
        try {
            runnable.execute();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    @FunctionalInterface
    public interface ThrowingSupplier<T> {
        T get() throws Exception;
    }

    @FunctionalInterface
    public interface ThrowingRunnable {
        void execute() throws Exception;
    }
}
```

*The post seemed too long so I split it in two. Second part is available [here](http://tidyjava.com/solving-small-problems-2).*