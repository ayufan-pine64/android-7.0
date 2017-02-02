# android-7.0
Main repository with Android 7.0 releases

## Build snapshot to speed-up build
export DIGITALOCEAN_API_TOKEN=<your-do-token>
packer build -only=digitalocean packer-template.json
