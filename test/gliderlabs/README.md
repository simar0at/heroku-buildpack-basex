# Usage

```bash
docker build --build-arg BUILDPACK_URL=https://github.com/simar0at/heroku-buildpack-basex -f $(pwd)/Dockerfile <some_test_app>
```

# Git DevOps environments

GitLab will pick this up if you set BUILDPACK_URL and use heroku based buildpacks.

Github can use this with the provided example workflow file.
