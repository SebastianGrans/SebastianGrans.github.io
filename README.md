# My Github Pages site

## Creating a new post

```bash
hugo new content content/posts/something.md
```

I tend to prepend them with dates, to make the `posts` less chaotic.

```bash
hugo new content content/posts/$(date "+%Y-%m-%d")-some-post-name.md  
```

It will create a file marked as `draft = true` which will not be published.

<https://gohugo.io/getting-started/quick-start/#add-content>

## Local development

Simply run:

```bash
hugo server
```

## Setup Linux

```bash
sudo apt install golang hugo
```

## Setup Windows

Install Go:

```bash
winget intsall GoLang.Go
```

Install Hugo extended:

```bash
winget install Hugo.Hugo.Extended
```
