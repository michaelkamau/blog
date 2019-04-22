---
title: Hello World
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

Some Rust Code Here:

```rust

use image::ColorType;
use image::png::PNGEncoder;
use num::Complex;
use std::str::FromStr;
use std::fs::File;
use std::io::Result;
use std::io::Write;

fn iterations_to_leave_set(c: Complex<f64>, limit: u32) -> Option<u32> {
    let mut z = Complex { re: 0.0, im: 0.0 };
    for i in 0..limit {
        z = z * z + c;
        if z.norm_sqr() > 4.0 {
            return Some(i);
        }
    }
    None
}

fn main() -> Result<()> {
    println!("Running Mandelbrot ...");
    mandelbrot_single()
}

fn parse_pair<T: FromStr>(s: &str, sep: char) -> Option<(T, T)> {
    match s.find(sep) {
        None => None,
        Some(index) => {
            let left = &s[..index].trim();
            let right = &s[index + 1..].trim();
            match (T::from_str(left), T::from_str(right)) {
                (Ok(l), Ok(r)) => Some((l, r)),
                _ => None,
            }
        }

    }
}

fn parse_complex(s: &str) -> Option<Complex<f64>> {
    match parse_pair(s, ',') {
        None => None,
        Some((re, im)) => Some(Complex { re: re, im: im }),
    }
}

#[test]
fn test_iterations_to_leave_set() {
    assert_eq![
        Some(0),
        iterations_to_leave_set(Complex { re: 2.1, im: 0.0 }, 1)
    ];

    assert_eq![
        None,
        iterations_to_leave_set(Complex { re: 2.0, im: 0.0 }, 1)
    ];
}

#[test]
fn test_parse_pair() {
    assert_eq!(parse_pair::<i32>("", 'r'), None);
    assert_eq!(parse_pair::<i32>("23", ','), None);
    assert_eq!(parse_pair::<i32>(",34", ','), None);
    assert_eq!(parse_pair::<i32>(",", ','), None);
    assert_eq!(parse_pair::<i32>("200,400", 'x'), None);
    assert_eq!(parse_pair::<i32>("200,400", ','), Some((200, 400)));
    assert_eq!(parse_pair::<i32>("200 , 400", ','), Some((200, 400)));
    assert_eq!(
        parse_pair::<f64>("200.34,400.45", ','),
        Some((200.34, 400.45))
    );
}

#[test]
fn test_parse_complex() {
    assert_eq!(
        parse_complex("1.0, 1.0"),
        Some(Complex { re: 1.0, im: 1.0 })
    );
    assert_eq!(parse_complex(",-0.0625"), None);
}


#[test]
fn test_pixel_to_point() {
    assert_eq!(
        pixel_to_point(
            (100, 100),
            (25, 75),
            Complex { re: -1.0, im: 1.0 },
            Complex { re: 1.0, im: -1.0 }
        ),
        Complex { re: -0.5, im: -0.5 }
    );
}

fn pixel_to_point(
    bounds: (usize, usize),
    pixel: (usize, usize),
    upper_left: Complex<f64>,
    lower_right: Complex<f64>,
) -> Complex<f64> {
    let (width, height) = (
        lower_right.re - upper_left.re,
        upper_left.im - lower_right.im,
    );
    Complex {
        re: upper_left.re + pixel.0 as f64 * width / bounds.0 as f64,
        im: upper_left.im - pixel.1 as f64 * height / bounds.1 as f64,
    }
}


fn render(
    pixels: &mut [u8],
    bounds: (usize, usize),
    upper_left: Complex<f64>,
    lower_right: Complex<f64>,
) {
    assert!(pixels.len() == bounds.0 * bounds.1);

    for row in 0..bounds.1 {
        for col in 0..bounds.1 {
            let point = pixel_to_point(bounds, (col, row), upper_left, lower_right);
            pixels[row * bounds.0 + col] = match iterations_to_leave_set(point, 255) {
                None => 0,
                Some(count) => 255 - count as u8,
            }
        }
    }
}

fn write_image(filename: &str,
pixels: &[u8], bounds: (usize, usize)) -> Result<()>{

    let output = File::create(filename)?;
    let encoder = PNGEncoder::new(output);
    encoder.encode(&pixels, bounds.0 as u32, bounds.1 as u32, ColorType::Gray(8))?;
    Ok(())
}


fn mandelbrot_single() -> Result<()> {

    let args: Vec<String> = std::env::args().collect();
    if args.len() != 5 {
        writeln!(std::io::stderr(),
        "Usage: mandelbrot FILE PIXELS UPPERLEFT LOWERRIGHT")
        .unwrap();

        writeln!(std::io::stderr(),
                 "Example: {} mandel.png 1000x750 -1.20,0.35 -1,0.20",
                 args[0])
            .unwrap();

        std::process::exit(1);
    }

     let bounds = parse_pair(&args[2], 'x')
        .expect("error parsing image dimensions");
    let upper_left = parse_complex(&args[3])
        .expect("error parsing upper left corner point");
    let lower_right = parse_complex(&args[4])
        .expect("error parsing lower right corner point");

    let mut pixels = vec![0; bounds.0 * bounds.1];

    render(&mut pixels, bounds, upper_left, lower_right);

    write_image(&args[1], &pixels, bounds)
        .expect("error writing PNG file");

    Ok(())   
}
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
