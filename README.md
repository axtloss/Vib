<div align="center">
    <img src="docs/brand/logo/svg/full-mono-dark.svg#gh-light-mode-only" height="64">
    <img src="docs/brand/logo/svg/full-mono-light.svg#gh-dark-mode-only" height="64">
    <p>Vib (Vanilla Image Builder) is a tool that streamlines the creation of container images. It achieves this by enabling users to define a recipe consisting of a sequence of modules, each specifying a particular action required to build the image. These actions may include installing dependencies or compiling source code. 
</p>
    <hr />
</div>

## Links

- [Website](https://vib.vanillaos.org/)
- [Examples](https://vib.vanillaos.org/examples)
- [Documentation](https://docs.vanillaos.org/collections/vib)

## Recipe Format

Highly inspired by the Flatpak manifest format.

A recipe is a YAML file that contains the image definitions, the commands to be executed during the build process and the list of modules to add resources to the image:

```yaml
base: debian:sid-slim
labels:
  maintainer: Vanilla OS Contributors
args:
  DEBIAN_FRONTEND: noninteractive
runs:
  - echo 'APT::Install-Recommends "0";' > /etc/apt/apt.conf.d/01norecommends
adds:
  rootfs.tar.gz: /
modules: []
```

### Description of Fields

- `base`: a string indicating the base image to use for the container image.
- `labels`: an object containing the labels to add to the container image.
- `args`: an object containing the build arguments to add to the container image.
- `runs`: an array of strings containing the commands to run during the build process.
- `modules`: an array of objects containing the modules to add to the container image.

## Module Structure

Each module specifies a specific step needed to build the image. Each module has the following format:

```yaml
name: module-name
type: module-type
path: modules-path
source:
  packages: module-source-packages
  path: module-source-path
  url: module-source-url
  type: module-source-type
  tag: module-source-tag
  commit: module-source-commit
buildflags: []
Buildvars: []
modules: {}
```

### Description of Fields

- `name`: a string representing the name of the module. This will be used as the identifier for the module so it must be unique.
- `type`: a string indicating the type of module. Currently, the supported types are "apt", "cmake", "dpkg", "dpkg-buildpackage", "go", "make", "meson" and "gen-modules".
- `path`: a string indicating the path where the modules to generate are located. Only used if type is "gen-modules".
- `source`: an object containing the information necessary to retrieve the source code for the module. The fields within this object depend on the module type.
  - `packages`: a string indicating the name of the package(s) to install, only used if type is "apt".
  - `paths`: a list of strings indicating the path to .deb files (or just a list of prefixes to search for .deb files) when used with"dpkg" or "dpkg-buildpackage", or a list of paths to .inst files which contain a list of packages to install when used with "apt".
  - `url`: a string indicating the URL to retrieve the source code, only required if source type is "tar" or "git".
  - `type`: a string indicating the type of source control system, currently "tar" and "git" are supported.
  - `tag`: a string indicating the git tag to use, only used if type is "git". Use either this or `branch` and `commit`.
  - `branch`: a string indicating the git branch to pull from, only used if type is "git". Use either this and `commit` or `tag`.
  - `commit`: a string with the commit hash to use, only used if type is "git". Passing "latest" to this field will use the most recent commmit from the branch specified above. Use either this and `branch` or `tag`.
- `buildFlags`: an array of strings indicating any additional build flags or options. Only used if type is "meson" or "go".
- `buildVars`: a collection of key-value pairs indicating any additional build variables. Only used if type is "go".
- `modules`: an array of objects containing the modules needed to build the source code.

## Includes

It is also possible to use the `includes.container` folder to include additional files in the container image. The files in this folder will be copied to the root of the container image and will be accessible during the build process and when the container is running.

The `includes.container` folder is located in the same directory as the recipe file and each file must be placed in a directory following the same structure in the Filesystem Hierarchy Standard (FHS). For example, if you want to include a file in the `/etc` directory, you would place it in the `includes.container/etc` folder.

### Module Execution Order

The module execution order is generated by prioritizing the modules dependencies. For example, if module A depends on module B, then module B will be executed before module A. If module A depends on module B which depends on module C, then module C will be executed first, then module B and finally module A.

So the following order:

```
module A
- module B
-- module C
```

Will be executed as:

```
module C
module B
module A
```

if any other module depends on module C or B, there is no need to specify the dependency again, since they are already in the execution order.

## Usage

To build an image using a recipe, you can use the `vib` command:

```
vib build recipe.yml
```

this will parse the recipe.yml to a Containerfile, which can be used to build the image with any container image builder, such as `buildah` or `podman`:

```
podman build -f Containerfile -t image-name
```
