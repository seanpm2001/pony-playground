A web interface for running Pony code.

# Running your own Pony-Playpen

## System Requirements

Currently needs to be run on a system with access to Docker.

## Running the web server

First, create the Docker image that playpen will use:

```
docker build docker -t ponylang-playpen
```

Get a github personal access token. Only the `gist` scope needs to be selected.  
Put it into the `GITHUB_TOKEN` environment variable. 

```
 export GITHUB_TOKEN="..."
```

It will be used for creating gists with the playgrounds contents.

Next, spin up the server.

```
cargo run --bin playpen
```

You should now be able to browse http://127.0.0.1:8000 and interact.

## Building for production

```
docker run --rm -it -v $HOME/.cargo/registry/ -v $PWD:/home/rust/src ekidd/rust-musl-builder:nightly-2020-09-04 cargo build --release
```
