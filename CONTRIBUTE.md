# How to contribute

If you want to contribute to **Sysadmin.cool** you can make some pull request in [this repository](https://github.com/algorys/sysadmin/pulls).

Be sure to follow the syntax of the posts and if possible offer a translation in French and English.

# Folder structure

Sysadmin.cool have a specific structure :

* tutoriel : contains any tutorial about GLPI, FusionInventory, Shinken, Alignak or Glpi-monitoring plugin.
* news : contains news about releases, features and other change in different projects.
* tool : contains other tutorial and tool for network administrators.

Each of the above folder contains a **en** and **fr** directory, which each contains a folder **_posts**. This last one contains articles.

* tag : this folder is used to generate each page corresponding to a tag.

If you want to add a tag, just make a corresponding folder and put an file call `index.md` inside. Add the following lines :

```conf
layout: mytag
key: your_key
```

Just change the key corresponding to your new tag. After, you have just to put it inside your new post.

# Writing a post

If you want to write a post, first decide if it's a news, a tutorial or simply an additionnal tool for you have to respect the following syntax in your head file :

## YAML additions

For example, if you make a post for how to update glpi, you'll need to add the following lines :

* For your french post :

```conf
lang: fr
ref: glpi-update
```

* For your english post :

```conf
lang: en
ref: glpi-update
```

As you can see, lang has to be change and the ref not. The `ref` is a **unique id** for your post to make possible switching between english and french. It's not the same like tag who can be assigned to different post.

Be sure your `ref` is **unique** !

Other YAML parameters are basic jekyll use. (See the [official documentation](https://jekyllrb.com/docs/posts/) for more detail).

## Naming conventions

Your articles, even if they are identical, must be named differently for both languages and respect the naming convention of jekyll :

For example :

```
YYYY-MM-DD-my-article-name.md
# and
YYYY-MM-DD-mon-nom-darticle.md
```

If they have the same `ref` (see below), the generator find alone the good traduction. I know it's a strange way to do that, but otherwise jekyll link to the same article everytime.

# Issues

The best way is to make it run on local before make a pull request and ensure `github-pages` is the last version.

If you can find a way to make it work properly, just open an issue in this repos.
