# U.S. Research Software Community Template

This is a community template for a USRSE group. It is provided for you
to copy to the [usrse GitHub](https://www.github.com/usrse) and then
have a portal for your center.

 - [How Does it Work?](#how-does-it-work) or specifically, what features does the page offer?
 - [Creating Your Community](#creating-your-community) comes down to creating a clone of the respository in the [usrse](https://www.github.com/usrse) GitHub organization.

## How does it work?

The site offers the following features:

 - A site wide search, and "Find an RSE" search to help users find support they need.
 - An optional Twitter feed if your group shares an alias.
 - Individual pages for your site members. Each individual can share a biography and areas of expertise to better offer support to users.

Additionally, all repositories in the usrse GitHub organization that begin with "community" serve
a json data structure that is used to search across the sites from a common page.
This means that any user can visit this page, and get directed to support at your institution.

## Creating your Community

You can create your repository from the [template here](https://github.com/USRSE/community-template/generate).
In order for it to show up in the USRSE "Find an RSE" search, you need to have
the template hosted at the [usrse](https://www.github.com/usrse/) GitHub organization,
so it's suggested that you become a member first.

### 1. About your Group

You can modify the [about.md](pages/about.md) file to add information about your
center. This will show up on the Community -> About tab.

### 2. Support Your Offer

In the [support.md](pages.support.md) you should describe the support that your
group offers. This will be searched by users to find help, so be sure to include
details about the resources and services that your group offers.

### 3. Add People

Any interested RSE at your institution can a page to the [_people](_people) 
collections folder. The header of the file should include links, social media,
and descriptors;

```yaml
---
name: Antonio T. Rex
title: Research Software Engineer
url: https://douglascuddletoy.com/shop/fantasy-whimsy/t-rex-s-dinosaur-w-sound/
image: rex.jpg
github: usrse
twitter: dinosaur
---
```

And they are free to write whatever they please in the content area! 
Images should be relative paths specified above, and expected to be in 
[assets/img/people](assets/img/people). A biography
and description of expertise is usually appropriate, as the content will be
searchable for users to find support for areas of interest.

### 4. Write Posts

If you want to use your community space to write articles or post news, you can
do so by adding a new markdown file to the [_posts](_posts) folder. We've left
a few entries there for you as examples. If you then want your post feed to
automatically appear on the [https://us-rse.org/blog](https://us-rse.org/blog)
site, your feed is located at the baseurl + `feed.xml`. So for this template, it
would be at https://localhost:4000/community-template/feed.xml.

### 5. Social Media

If your group has a Twitter alias, define the `twitter` variable in the [_config.yml](_config.yml)
file to show a feed of posts on the right side of the main page. By default this
is disabled, and individual engineers are able to define their own Twitter
aliases in their personal pages.

## Development

To develop the site, clone the repository and then build with jekyll:

```bash
bundle exec jekyll serve
```

### Important Notes

The "posts" page under the [posts](posts) folder is intentionally separate from
the other pages in [pages](pages) because the pagination plugin needs to find an
index.html to parse. Please don't rename or change the location of this file.

## Credit

The original template was from [bootstrap-clean-jekyll](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll) 
and has been simplified and updated for this case.
