# Docker for Mac: Docker Volume performance

## Problem

Baking a Docker image for a Ruby development environment works fine until you need to change your `Gemfile`.

Because `bundle install` is invoked within `docker build` it cannot be captured via a Docker volume, meaning it's locked inside the image layer.

This means the next time we change our `Gemfile` we have to run `docker build` again, which runs `bundle install` as though it's the first time again.

## Solution

The obvious solution is to skip baking an image and instead use Docker Compose to define the development environment on top of the official `ruby` image.

This approach allows us to mount a Docker volume over `$GEM_HOME` and run `bundle install` via Compose so we can capture the result.

Next time we change our `Gemfile` and run `bundle install`, Bundler will see the existing `$GEM_HOME` and only install gems which have changed.

## Problem

The way Ruby walks the `$GEM_PATH` to load files via a `require` statement doesn't line up with Docker for Mac's performance when traversing a mounted volume.

The result is that we've saved heaps of time when we need to run `bundle install`, but any other command which loads a large number of files (such as Rails) is horribly slow.

## Bootstrap

```
docker-compose -f docker-compose.yml -f docker-compose.build.yml build
docker-compose -f docker-compose.yml -f docker-compose.image.yml run --rm rails bundle install
```

## Benchmark:

`bin/rake environment` can be used to load the Rails app and `bin/rails test` invokes Minitest, although there are no tests to run.

You can opt-in to the 'bake-gems-into-an-image' or 'mount-gems-in-a-volume' approaches by loading `docker-compose.build.yml` and `docker-compose.image.yml` respectively.

This gives us four commands necessary to run the benchmark:

```
time docker-compose -f docker-compose.yml -f docker-compose.build.yml run --rm rails bin/rake environment > /dev/null
time docker-compose -f docker-compose.yml -f docker-compose.image.yml run --rm rails bin/rake environment > /dev/null
time docker-compose -f docker-compose.yml -f docker-compose.build.yml run --rm rails bin/rails test > /dev/null
time docker-compose -f docker-compose.yml -f docker-compose.image.yml run --rm rails bin/rails test > /dev/null
```

## Results

All results are averaged across three attempts.

### Docker for Mac

Hardware: Early 2015 13-inch Retina MacBook Pro, 3.1 GHz i7 CPU, 16 GB RAM

| | bin/rake environment | bin/rails test |
| --- | --- | --- |
| **Build** | 4.162 sec | 2.711 sec |
| **Image** | 39.335 sec | 14.420 sec |

### Dinghy in VirtualBox

Hardware: Early 2015 13-inch Retina MacBook Pro, 3.1 GHz i7 CPU, 16 GB RAM

| | bin/rake environment | bin/rails test |
| --- | --- | --- |
| **Build** | 2.001 sec | 1.198 sec |
| **Image** | 3.625 sec | 1.302 sec |

### Docker on Ubuntu 16.04

Hardware: Intel NUC, 1.8 GHz i5 6260U, 8 GB RAM

| | bin/rake environment | bin/rails test |
| --- | --- | --- |
| **Build** | 1.902 sec | 1.237 sec |
| **Image** | 1.772 sec | 1.176 sec |
