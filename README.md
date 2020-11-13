# Petrol Minimalist Jekyll Template

Live demo at https://rasmusgs.com/petrol-jekyll-theme

This is a simple and minimalist template for Jekyll for those who likes to eat cake :cake:.

Theme now supports **dark mode**! See screenshot below, or try it in a browser supporting dark mode, e.g. Chrome or Safari. Dark colors are of course customizable.

Theme forked from [kopplin](https://github.com/sergiokopplin/indigo), with social links inspired by [murraco](https://github.com/murraco/jekyll-theme-minimal-resume).

***

<p align="center">
    <b><a href="README.md#what-is-inside">What is inside</a></b>
    |
    <b><a href="README.md#setup">Setup</a></b>
    |
    <b><a href="README.md#settings">Settings</a></b>
    |
    <b><a href="README.md#how-to">How to</a></b>
</p>

<p align="center">
    <img src="https://raw.githubusercontent.com/rasgs/petrol-jekyll-theme/gh-pages/assets/screen-shot.png" />
    <img src="https://raw.githubusercontent.com/rasgs/petrol-jekyll-theme/gh-pages/assets/screen-shot-dark.png" />
</p>

## What is inside

- [Jekyll](https://jekyllrb.com/), [Sass](https://sass-lang.com/) ~[RSCSS](https://rscss.io/)~ and [Font-Awesome](https://fontawesome.com/);
- Tests with [Travis](https://travis-ci.org/);
- Google Speed: [98/100](https://developers.google.com/speed/pagespeed/insights/?url=https%3A%2F%2Frasmusgs.com%2Fpetrol-jekyll-theme%2F);
- No JS. :sunglasses:

## Setup

0. :star: to the project. :metal:
1. Fork the project [Petrol](https://github.com/rasgs/petrol-jekyll-theme)
2. Edit `_config.yml` with your data (check <a href="README.md#settings">settings</a> section)
3. Write some posts :bowtie:

If you want to test locally on your machine, do the following steps also:

1. Install [Jekyll](https://jekyllrb.com), [NodeJS](https://nodejs.org/) and [Bundler](https://bundler.io/).
2. Clone the forked repo on your machine
3. Enter the cloned folder via terminal and run `bundle install`
4. Then run `bundle exec jekyll serve --config _config.yml,_config-dev.yml`
5. Open it in your browser: `http://localhost:4000`
6. Do you want to use the [jekyll-admin](https://jekyll.github.io/jekyll-admin/) plugin to edit your posts? Go to the admin panel: `http://localhost:4000/admin`. The admin panel will not work on GitHub Pages, [only locally](https://github.com/jekyll/jekyll-admin/issues/341#issuecomment-292739469).

## Settings

You must fill some informations on `_config.yml` to customize your site.

```
name: John Doe
bio: 'A Man who travels the world eating noodles'
picture: 'assets/images/profile.jpg'
...

and lot of other options, like width, projects, pages, read-time, tags, related posts, animations, multiple-authors, etc.
```

Colors can be customized in `_sass -> base -> variables.sass`:

```
// theme main color (petrol blue)
$delta: #2E93A4 !default

// subtitle color
$gamma: #999

\:root
  // headlines
  --page-title: #222
  // main text
  --page-main: #666
  // accents
  --epsilon: #f0f0f0

/* Light mode */
@media (prefers-color-scheme: light)
  \:root
      --page-title: #222
      --page-main: #666
      --epsilon: #f0f0f0
  body
      background-color: white

/* Dark mode */
@media (prefers-color-scheme: dark)
  \:root
      --page-title: #ddd
      --page-main: #bbb
      --epsilon: #333

  body
      background-color: #151515

  img
    opacity: .75
    transition: opacity .5s ease-in-out


  img:hover
    opacity: 1

```


## How To?

Check the [FAQ](./FAQ.md) if you have any doubt or problem.

---
## License

[MIT](https://github.com/rasgs/petrol-jekyll-theme/blob/gh-pages/LICENSE) License © Rasmus Gundorff Sæderup
