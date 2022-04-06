---
layout: post
title:  "How to Publish Markdowns to Medium"
date:   2022-04-04 14:00:00 +0100
categories: guide medium
crosspost_to_medium: true
---

# Guide - Using Plain Text, Git and Automation to Publish Medium Stories
Dear reader, welcome to my first guide. Recently I started looking for a new way of learning. That's how I ended up at Medium, but now as a writer. In the past I have often used the platform as a reference book for learning. I've decided to take it to another level and challenge myself as a writer, as I hope to contribute to the community by writing and to also learn more about the topics covered.

At least I can say that I am a fan. Unfortunately, Medium doesn't offer any options to write in Markdown, let alone automate publishing from a Git repository. So in this guide I will explain what I think is the best option for publishing plaintext Markdown as stories to Medium and guide you through it.

## Why Do I Want This?
My inspiration for this is an extension of the documentation as code (docs-as-code) approach, which refers to the idea that you should be writing documentation with the same tools as code - if you are a developer who is writing technical documentation.

**Markdown** is a lightweight markup language that you can use to add formatting elements to plaintext documents. It gets rid of all distractions of formatting that traditional text editors have (*e.g.* Microsoft Word). It allows you to format your writing without taking your fingers off the keyboard. As such, your writing experience becomes seamless without losing focus on what's important (*i.e.* content). And a big plus; since it's plaintext it's also platform independent.

**Git** is a version control system that is used by developers all over the world. It will help to track different versions of Markdowns, and to review texts with diffs and comments and collaborate with other writers. And, most of the technical documentation that can be found on [GitHub](https://medium.com/r/?url=https%3A%2F%2Fgithub.com) is already written in Markdown – there are plenty of examples.

**Automation** is important, because writing in Markdown alone and using some copy paste solution will not suffice. We need a way of importing Markdown texts into Medium stories as seamless as writing the Markdowns and pushing them to a Git repository. Therefore, publishing a story to Medium should be a part of this process. Because a good developer is a lazy developer! That is, most developers hate doing repetitive tasks and one way to avoid this is by introducing automation.

### The General Idea
We will be using [GitHub Pages](https://pages.github.com) with [Jekyll](https://jekyllrb.com) in this guide. This allows us to transform plain text into static websites and blogs. GitHub Pages is powered by Jekyll, so we can easily deploy our site using GitHub for free - we can even use a [custom domain name](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages). As for automation, we will use the [jekyll-crosspost-to-medium](https://github.com/aarongustafson/jekyll-crosspost-to-medium) plugin to automatically syndicate our posts to Medium from our Jekyll blog. We will combine this with [GitHub Actions](https://docs.github.com/en/actions) to automate the process after a merge to the main branch for the GitHub Page. 

![general_idea](/assets/images/general_idea.png)

So the general idea is to write stories in Visual Studio Code, push changes to the GitHub repository and trigger a publish on GitHub Pages and Medium - preferably after a merge.

## Prerequisites
1. You're familiar with version control systems such as Git, or [start learning](https://git-scm.com).
2. You've created an account on GitHub, or [sign up](https://github.com).
3. You know how to write in Markdown, or [start learning](https://www.markdownguide.org/getting-started/).
4. You've installed a text editor, or download [Visual Studio Code](https://code.visualstudio.com).

## Let's Get Started!

### 1. Setup GitHub Pages
This step is pretty easy and can be achieved by following the basic guide that [GitHub Pages](https://pages.github.com) offers. Just read everything carefully and do not deviate from the steps they explain in the guide. If you've done so successfully then the end result will be a skeleton website with an *index.html* that shows "Hello World" on your personal page: **https://username.github.io**.  We can ignore this *index.html* file for now, because we will be writing in Markdown and not HTML. 
Open up a terminal (or command line) on your local machine and clone in your git repository and replace USERNAME with your own: 
```sh
git clone https://github.com/USERNAME/USERNAME.github.io.git
```

Open the newly created folder with:
```sh
cd USERNAME.github.io
```

If you choose to publish your site from the `gh-pages` branch, create and checkout the `gh-pages` branch with:
```sh
git checkout - orphan gh-pages
```

You could decide to use another source, you can read more about that [here](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site). Inside the current directory you'll see the *index.html*, delete this with:

```sh
rm -rf index.html
```

Now we're ready to setup Jekyll - but, leave the terminal open with the current directory.

> ***N.B.** You can skip the [Blogging with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll) guide that GitHub Pages offers, since we will be using the (superior) installation guide that Jekyll offers in **2. Jekyll Installation** instead. But, feel free to use the GitHub Pages guide as a reference if needed, of course!*

### 2. Jekyll Installation
Jekyll converts Markdown to HTML using the Markdown filter specified in a config file. You can read about the order of interpretation [here](https://jekyllrb.com/tutorials/orderofinterpretation/). This step is a bit more tricky. But, Jekyll offers an in-depth [setup guide](https://jekyllrb.com/docs/) for each OS. So if you follow the guide correctly, then on MacOS you'll end up installing:
1. **Ruby**, the programming language that is used to power Jekyll. 
2. **Bundler**, the Ruby gem which is used to manage the application's gems for your project - somewhat similar to NPM; it uses files called `Gemfile` and `Gemfile.lock`.
3. **Jekyll**, the static site generator.

Before you proceed with the next step, make sure that your environment settings are configured correctly by running `gem env` in your terminal.

### 3. Setup a Jekyll Site
Now we're ready to setup a Jekyll site. That can be achieved easily by running the following command in the terminal inside the *username.github.io* directory:
```sh
jekyll new --skip-bundle .
```

After running this command all the necessary starter files and *_posts* directory will be generated for you. Now, fire up **Visual Studio Code (VS Code)** and import your *username.github.io* folder. You'll see the following directory structure in your explorer:
![vscode](/assets/images/vscode.png)

As you can see in the image above, VS Code offers a preview mode for Markdown editing. By default, Markdown previews automatically update to show the currently active *.markdown* file.

If you're using **zsh**, it might be that you still need to adjust your gem path - if this error occurs: `zsh: command not found: jekyll`. Try to add the following and replace the `X.X` with the first two numbers of the Ruby version you're using:
```sh
export PATH="$HOME/.gem/ruby/X.X.0/bin:$PATH"
```

And then reinstall Bundler and Jekyll with:
```sh
gem install --user-install bundler jekyll
```

> ***N.B.** If you're using different versions of Ruby there is a tool called **rbenv** which allows you to install and run different versions of Ruby side-by-side. This is also explained in the Jekyll documentation. So, this and your environment variables could be something to look out for while you're troubleshooting.*

### 4. Configuring your Site
Open the *Gemfile* that Jekyll created in VS Code and follow the steps below for a basic configuration:

1. Add "#" to the beginning of the line that starts with `gem "jekyll"` to comment out this line - or delete it entirely if you're certain you won't need it. 
2. Delete the "#" from the beginning of `# gem "github-pages"` and change this line to `gem "github-pages", "~> VERSION", group: :jekyll_plugins`. Replace VERSION with the latest supported version of the `github-pages` gem. You can find this version [here](https://pages.github.com/versions/). Save and close the *Gemfile*.
3. From the terminal, run `bundle install`. If this goes well, you'll see an output in the terminal similar to this:
```sh
Bundle complete! 7 Gemfile dependencies, 101 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

You can see that several *Gemfile* dependencies were created, which are locked in by a file called *Gemfile.lock* that was created at the root of your *username.github.io* directory. This lock-file gets created when you run `bundle install` for the first time. And so, when you have the same repository checked out and you run `bundle install` on a different machine, Bundler will find the `Gemfile.lock` and skip the dependency resolution step. Instead, it will install all of the same gems that you used on the original machine (*i.e.* we don't have to guess which versions of the dependencies we should install). 

Now it's time to test if everything works correctly! Run your Jekyll site locally with:
```sh
bundle exec jekyll serve
```

This will run your site on http://localhost:4000 by default, but you can change this by adding a different port in *your_config.yml* file by just adding the following line:
```sh
port: 4001
```

### 5. Create a Blog Post
When you run your site with `bundle exec jekyll serve` you'll see "Posts" on the Home screen. Here, you can also find an example post called "Welcome to Jekyll!". You can find this post and its details in the *_posts* directory that Jekyll created for you at the root of your *username.github.io* directory. Open up the file in VS Code. At the top of the file you'll find what's called the [front matter](https://jekyllrb.com/docs/front-matter/) of your post:
```
---
layout: post
title:  "Welcome to Jekyll!"
date:   2022-03-19 21:49:12 +0100
categories: jekyll update
---
```

You can see that for this post the front matter is used to categorize it into two categories. To understand what's happening under the hood, we'll have to take a look at the directory tree for the generated *_site* folder:
```sh
.
├── 404.html
├── about
│   └── index.html
├── assets
│   ├── main.css
│   └── minima-social-icons.svg
├── feed.xml
├── index.html
└── jekyll
    └── update
        └── 2022
            └── 03
                └── 19
                    └── welcome-to-jekyll-copy.html
```

Categories of a post may be incorporated into [the generated URL](https://jekyllrb.com/docs/permalinks/) for the post. So if your front matter has `category: jekyll update`, or `categories: jekyll update` the post would have the URL as either `../jekyll%20update/2022/03/19/..` or `../jekyll/update/2022/03/19/..` respectively - as you can see, this is represented in your directory tree. You can read more about Jekyll posts [here](https://jekyllrb.com/docs/posts/).

For your site you can copy this *.markdown* file and replace it's content with your own. Make sure to adjust the filename and front matter according to your needs.

### 6. Install the Jekyll Crosspost Plugin
In this next step we'll be installing the [crosspost plugin](https://github.com/aarongustafson/jekyll-crosspost-to-medium) that Aaron Gustafson created for Medium. Follow the steps explained in his setup. For a plugin installation guide it's best to follow [Jekyll's installation guide](https://jekyllrb.com/docs/plugins/installation/), and add the plugin as a gem to your *Gemfile*, like so:
```lock
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-crosspost-to-medium"
end
```

And make sure that you update the *Gemfile.lock* after by running:
```sh
bundle install
```

Make sure you add the environment variables from the crosspost guide to your local machine as explained in the installation guide and in the [GitHub environment secrets](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#environment-secrets). Like so: 
![env-vars](/assets/images/repo-vars.png)

### 7. Publishing to Medium
Now we can add the `$PLACEHOLDER` value to `crosspost_to_medium` in the front matter of our post so that we can crosspost to Medium: 
```
---
layout: post
title:  "Welcome to Jekyll!"
date:   2022-03-19 21:49:12 +0100
categories: jekyll update
crosspost_to_medium: $PLACEHOLDER
---
```

We will set it to a placeholder value, because we don't want a publishing to be triggered by the `bundle exec jekyll serve` command when we are testing the Jekyll site locally. This is going to be handled with [GitHub actions](https://github.com/features/actions) after a merged is made to the master. We will do this by adding a Jekyll workflow with specific actions in our repository. Again, the [Jekyll documentation](https://jekyllrb.com/docs/continuous-integration/github-actions/) for GitHub actions comes in handy, and we'll use their *.yaml* file as a base for:
```yaml
name: Build and deploy Jekyll site to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  publish-to-medium:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set crosspost_to_medium in all .markdown to true
        run: |
          sed -i 's/crosspost_to_medium: $PLACEHOLDER/crosspost_to_medium: true/g' _posts/*.markdown
      - uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Apply IS_CROSSPOST to True

  github-pages:
    needs: [publish-to-medium]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - uses: helaili/jekyll-action@v2 # this handles the build and deploy
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
        env:
          MEDIUM_INTEGRATION_TOKEN: ${{ secrets.MEDIUM_INTEGRATION_TOKEN }}
          MEDIUM_USER_ID: ${{ secrets.MEDIUM_USER_ID }}
```
As shown in the code block above, an extra job replaces the placeholder value in the *.markdown* files to `crosspost_to_medium: true`. We use an action created by *stefanzweifel* to commit the changes we make to the files. After this initial job the Jekyll site will be built and deployed with the `github-pages` job. An extra line (*i.e.* `env:`) is added to set the environment variables that are needed for the crosspost plugin.

> ***N.B.** You can just set the front matter variable to `crosspost_to_medium: true` and then serve your Jekyll site - so that it will publish your posts to Medium at once. But, making it part of the workflow process will give you more control over when to publish to Medium. You can for example also choose to publish when you create a release.* 

## Conclusion
That's it! You're all set. Meaning, after a pull request gets merged in your GitHub repository - that holds your Jekyll site, a crosspost to Medium will be triggered by the actions that were added to the GitHub workflow. This post was created using this setup, you can find the source code [here](https://github.com/selaianwary/selaianwary.github.io). I look forward to your feedback! 