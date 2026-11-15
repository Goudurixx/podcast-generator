# podcast-generator

A GitHub Action that turns a YAML file into a podcast RSS feed (`podcast.xml`) and commits it back to the repository.

> Course exercise. Built to learn how podcast feeds and GitHub Actions fit together. It is not a production tool.

## What it does

1. Reads `feed.yaml` (podcast metadata and a list of episodes) from the repository root.
2. Writes an RSS 2.0 file with the iTunes namespace to `podcast.xml`.
3. Commits the result as `🤖 : Udpate Feed` and pushes it to `main`.

Audio files and artwork stay in the repo. Every enclosure and image URL is built as `link` + `file` (or `image`) from `feed.yaml`, so the feed works once the repository is served over HTTP, for example with GitHub Pages.

## How the Action works

- `action.yml` declares a Docker action (`runs: using: docker`) with two inputs, `name` and `email`, which default to the workflow actor.
- `Dockerfile` builds a small image (Ubuntu, Python, PyYAML, git) and copies in `feed.py` and `entrypoint.sh`.
- `entrypoint.sh` sets the git identity, runs `feed.py` in the mounted workspace, then runs `git add -A`, commits and pushes.
- `feed.py` parses the YAML with PyYAML and builds the XML with `xml.etree.ElementTree`.

## Feed format

```yaml
title: My Podcast
subtitle: Short subtitle
author: Author Name
description: What the show is about.
image: /images/artwork.jpg
language: en-us
category: Technology
format: audio/mpeg
link: https://<user>.github.io/<repo>/
item:
  - title: EP01-First episode
    description: What this episode covers.
    published: Thu, 12 Jan 2023 18:00:00 GMT
    file: /audio/EP01.mp3
    duration: 00:00:36
    length: 576324
```

`length` is meant to be the file size in bytes. See the limitations below.

## Usage

Add a workflow to the repository that holds `feed.yaml`:

```yaml
name: Generate Podcast Feeds
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repo
        uses: actions/checkout@v3
      - name: Run Feed Generator
        uses: Goudurixx/podcast-generator@main
```

The default `GITHUB_TOKEN` must be allowed to push (repository Settings > Actions > General > Workflow permissions). Serve the repo with GitHub Pages to publish the feed.

To run the script locally without the Action:

```bash
pip install pyyaml
python3 feed.py   # reads feed.yaml, writes podcast.xml in the current directory
```

## Limitations

- The push target is hard-coded to `main` in `entrypoint.sh`.
- The Dockerfile uses `ubuntu:latest` and installs `python3.10` without pinning. I have not verified that the image builds on a current Ubuntu.
- `length` is copied as written. A value such as `576,324` (with a comma) is not a valid byte count for RSS. Use plain integers.
- There is no input validation: a missing key in `feed.yaml` fails with a Python traceback.
- The bot's commit triggers the `on: [push]` workflow again. The second run should find nothing to change, but I have not checked that.
- No tests.

## Credits

Written as an exercise. The example feed used during testing is [podcast-provider](https://github.com/Goudurixx/podcast-provider), a feed for *The Future in Tech* by Ray Villalobos (LinkedIn Learning). The podcast content in that repo is not mine.

## License

MIT, see [LICENSE](LICENSE).
