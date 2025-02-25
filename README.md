# Resize

Image resizing for the [Go](http://go.dev) with common interpolation methods.
This fork aims to continue maintaining the project after it was archived upstream.

## Installation

Import into your Go module with the following command:

```bash
go get github.com/rymdport/resize
```

## Usage

This package needs at least Go 1.19. Import package with:

```go
import "github.com/rymdport/resize"
```

The resize package provides 2 functions:

* `resize.Resize` creates a scaled image with new dimensions (`width`, `height`) using the interpolation function `interp`.
  If either `width` or `height` is set to 0, it will be set to an aspect ratio preserving value.
* `resize.Thumbnail` downscales an image preserving its aspect ratio to the maximum dimensions (`maxWidth`, `maxHeight`).
  It will return the original image if original sizes are smaller than the provided dimensions.

```go
resize.Resize(width, height uint, img image.Image, interp resize.InterpolationFunction) image.Image
resize.Thumbnail(maxWidth, maxHeight uint, img image.Image, interp resize.InterpolationFunction) image.Image
```

The provided interpolation functions are (from fast to slow execution time)

- `NearestNeighbor`: [Nearest-neighbor interpolation](http://en.wikipedia.org/wiki/Nearest-neighbor_interpolation)
- `Bilinear`: [Bilinear interpolation](http://en.wikipedia.org/wiki/Bilinear_interpolation)
- `Bicubic`: [Bicubic interpolation](http://en.wikipedia.org/wiki/Bicubic_interpolation)
- `MitchellNetravali`: [Mitchell-Netravali interpolation](http://dl.acm.org/citation.cfm?id=378514)
- `Lanczos2`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=2
- `Lanczos3`: [Lanczos resampling](http://en.wikipedia.org/wiki/Lanczos_resampling) with a=3

Which of these methods gives the best results depends on your use case.

Sample usage:

```go
package main

import (
	"github.com/rymdport/resize"
	"image/jpeg"
	"log"
	"os"
)

func main() {
	// open "test.jpg"
	file, err := os.Open("test.jpg")
	if err != nil {
		log.Fatal(err)
	}

	// decode jpeg into image.Image
	img, err := jpeg.Decode(file)
	if err != nil {
		log.Fatal(err)
	}
	file.Close()

	// resize to width 1000 using Lanczos resampling
	// and preserve aspect ratio
	m := resize.Resize(1000, 0, img, resize.Lanczos3)

	out, err := os.Create("test_resized.jpg")
	if err != nil {
		log.Fatal(err)
	}
	defer out.Close()

	// write new image to file
	jpeg.Encode(out, m, nil)
}
```

## Caveats

* Optimized access routines are used for `image.RGBA`, `image.NRGBA`, `image.RGBA64`, `image.NRGBA64`, `image.YCbCr`, `image.Gray`, and `image.Gray16` types. All other image types are accessed in a generic way that will result in slow processing speed.
* JPEG images are stored in `image.YCbCr`. This image format stores data in a way that will decrease processing speed. A resize may be up to 2 times slower than with `image.RGBA`. 


## License

Copyright (c) 2012 Jan Schlicht <janschlicht@gmail.com>.
Resize is released under a MIT style license.
