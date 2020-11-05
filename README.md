# cyber

Jekyll theme for Github Pages

[Demo]( https://jayeeeeee.github.io/ )

## Features

1. ScrollSpy
2. Summary Chart
3. Google Analytic

## Installation

**Copy and Paste** to your Github Pages `root` folder

## Usage

### Run locally

```shell
$ bundle exec jekyll serve
```

Known Issue

```shell
Conversion error: Jekyll::Converters::Scss encountered an error while converting 'assets/css/style.scss':
                  Invalid CP950 character "\xE2" on line 5
           Error: Invalid CP950 character "\xE2" on line 5
```

-> add new file `assets/css/style.scss`:

```text
---
---

@charset "UTF-8";
```

### Configuration

`_config.yml`

- title
- description
- url
- google_analytics

### Logo

Replace `/assets/logo.png`

## Dependencies

- Bootstrap - <https://getbootstrap.com/>
- jekyll-toc - <https://github.com/allejo/jekyll-toc>
- Google Fonts - <https://fonts.google.com/>
- Chart.js - <https://www.chartjs.org/>

## Reference

Jekyll Theme - <https://jekyllrb.com/docs/themes/#creating-a-gem-based-theme>

Github Pages : Dependency versions - <https://pages.github.com/versions/>

## License

The theme is available as open source under the terms of the [MIT License](/LICENSE.txt).
