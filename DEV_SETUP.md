## Development setup — Society API Docs

This document explains how to set up, configure, and run the Society API Docs project for local development. It covers native development on Windows/macOS/Linux and using the included Vagrant VM.

### Quick summary / contract

- Inputs: a machine with internet access.
- Outputs: a running Middleman preview server at http://localhost:4567 and a generated static site in `build/` after running the build.
- Success criteria: `bundle exec middleman server` serves the docs on port 4567; `./deploy.sh` can build and push the `build/` directory to the `gh-pages` branch.

## Prerequisites

- Git (to clone the repo).
- Ruby — the project requires Ruby >= 2.6. The project `Gemfile.lock` records Ruby 2.7.2. Prefer installing Ruby 2.7.x to match the lockfile.
- Bundler (bundled with modern RubyInstaller or installed via gem).
- Node.js (some build tools depend on Node; the Vagrant provisioning installs Node 12).

Windows-specific notes

- On Windows, use RubyInstaller (https://rubyinstaller.org/) with MSYS2 (DevKit). Install Ruby 2.7.x (recommended: 2.7.2 to match Gemfile.lock). During the RubyInstaller steps run the MSYS2 installer and the recommended development toolchain (ridk install).
- Make sure the ruby/bin and gem/bin are on your PATH.
- If you prefer to avoid a native Ruby install on Windows, use the Vagrant flow described below.

## Recommended versions (from repo)

- Ruby: 2.7.2 (Gemfile.lock)
- Bundler: 2.4.22 (Gemfile.lock)
- Middleman: ~> 4.4 (Gemfile)

## Setup (native machine)

1. Open a terminal (PowerShell on Windows).
2. Clone the repository (if you haven't already):

   git clone <repo-url>
   cd society-api-docs

3. Install bundler (if not already installed) and install gems:

   gem install bundler -v 2.4.22 --no-document
   bundle config build.nokogiri --use-system-libraries
   bundle install

Notes:

- The project depends on native extensions (nokogiri). On Windows you should have MSYS2/DevKit available (RubyInstaller handles this). On Linux/macOS, make sure libxml2/libxslt and build-essential tools are installed.

## Run the development server (native)

1. From the project root run:

   bundle exec middleman server

2. Open your browser at http://localhost:4567

Options:

- If the file watcher is unreliable (especially on Windows/VMs), use the watcher options shown in the Vagrantfile:

  bundle exec middleman server --watcher-force-polling --watcher-latency=1

Middleman will read `config.rb` which configures markdown, assets, relative links, and sets the server port (4567).

## Build the static site

To generate static files into `build/`:

bundle exec middleman build --clean

This will run the build configuration in `config.rb` (asset hashing, minification) and output HTML/CSS/JS into `build/`.

## Deploy (generate + push to gh-pages)

There is a `deploy.sh` script that automates building and pushing the `build/` directory to the `gh-pages` branch. Basic usage from the repo root:

./deploy.sh

Common options are described in the script help (run `./deploy.sh -h`).

Notes for Windows users:

- `deploy.sh` is a POSIX shell script. Run it from WSL, Git Bash, or a Unix-like environment. Alternatively, run the build with `bundle exec middleman build --clean` and push `build/` to `gh-pages` manually.

## Vagrant (recommended for Windows or if you want a reproducible Linux dev environment)

The included `Vagrantfile` provisions an Ubuntu 20.04 VM and attempts to install Ruby, Node, and bundler, then runs the Middleman server on boot.

1. Install VirtualBox and Vagrant.
2. From the repo root run:

   vagrant up

3. The Vagrantfile forwards port 4567 from the guest to the host, so after provisioning the site should be available at http://localhost:4567 on your host machine.

What the Vagrant provisioning does (high level):

- Installs Ruby and build tools.
- Installs Node.js v12.
- Installs bundler and runs `bundle install` inside the VM.
- Starts the Middleman server in the background and logs output to `~/middleman.log` in the VM.

How to access the VM for debugging:

vagrant ssh

# inside the vm

cd /vagrant
tail -f ~/middleman.log

To stop the background middleman process in the VM, find and kill it with `ps`/`kill` or `pkill -f middleman`.

## Troubleshooting

- If gem installation of `nokogiri` fails, install system libraries (libxml2-dev, libxslt-dev) and re-run `bundle install`. On Windows ensure MSYS2/DevKit is installed and updated.
- If Middleman hangs while building or serving, try disabling minify/asset_hash in `config.rb` temporarily or run the server with `--watcher-force-polling`.
- If you see port conflicts, check processes using port 4567 and either kill them or change the port by running `bundle exec middleman server --port=XXXX`.

## Additional notes and follow-ups

- Consider adding a simple `Makefile` or `rake` tasks to standardize commands (server, build, deploy).
- CI: Add a simple CI job that runs `bundle install` and `bundle exec middleman build --clean` to validate PRs.

## Quick commands reference

- Install gems (recommended bundler version pinned):

  gem install bundler -v 2.4.22 --no-document
  bundle config build.nokogiri --use-system-libraries
  bundle install

- Run server (dev):

  bundle exec middleman server

- Build (prod):

  bundle exec middleman build --clean

- Deploy using script:

  ./deploy.sh

---

Created from the repository's `Gemfile`, `Gemfile.lock`, `config.rb`, `Vagrantfile`, and `deploy.sh`.
