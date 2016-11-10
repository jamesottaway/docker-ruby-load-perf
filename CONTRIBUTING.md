## Benchmark Result Template

Copy the following template and fill in the required sections:

```
### Hardware

e.g: Early 2015 13-inch Retina MacBook Pro, 3.1 GHz i7 CPU, 16 GB RAM

### docker info

Run `docker info` and paste the output below:

    Containers: 62
     Running: 6
    Images: 370
    Server Version: 1.12.3
    … etc.

### Results

Run the following commands and paste the output:

    time docker-compose -f compose.yml -f docker.build.yml run --rm rails bin/rake environment > /dev/null
    time docker-compose -f compose.yml -f docker.image.yml run --rm rails bin/rake environment > /dev/null
    time docker-compose -f compose.yml -f docker.build.yml run --rm rails bin/rails test > /dev/null
    time docker-compose -f compose.yml -f docker.image.yml run --rm rails bin/rails test > /dev/null
```
