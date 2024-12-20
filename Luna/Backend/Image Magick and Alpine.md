# Image Magick, WKHTMLTOPDF,  and Alpine

> [!Important]
> The deal with Alpine is we need version 3.14 because is the latest version supporting `wkhtmltopdf`.

Before AppBlend, the Image Magick version in Edge was **7.1.0-50 beta Q16** and for Patient Self Report it was **7.0.11-14 Q16**.

In general, the whole combination of Image Magick, Ruby and Alpine was like this:

- Edge
	- Ruby 3.0.6
	- Alpine 3.16
	- Image Magick 7.1.0
- Patient Self Report
	- Ruby 3.1.0
	- Alpine 3.14
	- Image Magick 7.0.11

After migrating Edge to Ruby 3.1.0, with Alpine 3.14, Image Magick *was downgraded* to the same version as in Patient Self Report.

To date, these are the combination of docker images for Ruby 3.1.0 with Alpine:

- 3.1.0-alpine3.15
- 3.1.0-alpine3.14
- 3.1.0-alpine

Se here https://hub.docker.com/_/ruby/tags?page=&page_size=&ordering=&name=3.1.0-alpine

## What's the deal with this combination?

The important thing here is that ==the minimum version of Alpine with support for WKHTMLPDF is 3.14==.

So we need to carefully consider this every time we want to upgrade the Ruby version.

In Alpine packages website only [3.14](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.14&repo=&arch=&maintainer=) lists results for this software.