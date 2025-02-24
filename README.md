# Pipenv Install Cloud Native Buildpack
The Paketo Pipenv Install Buildpack is a Cloud Native Buildpack that installs
packages using pipenv and makes it available to the application.

The buildpack is published for consumption at
`gcr.io/paketo-buildpacks/pipenv-install` and `paketobuildpacks/pipenv-install`.

## Behavior
This buildpack participates if `Pipfile` exists at the root the app.

The buildpack will do the following:
* At build time:
  - Installs the application packages to a layer made available to the app.
  - Prepends the layer site-packages onto `PYTHONPATH`.
  - Prepends the layer's `bin` directory to the `PATH`.
* At run time:
  - Does nothing

This buildpack speeds up the build process by reusing (the layer of) installed
packages from a previous build if it exists, and later cleaning up any unused
packages. For apps that do not have a `Pipfile.lock`, clean-up is not performed
to avoid the overhead of generating a lock file. Users of such apps should
either include a lock file with their app, or clear their build cache during a
rebuild to avoid any unused packages in the built image.

## Integration

The Pipenv Install CNB provides `site-packages` as a dependency. Downstream
buildpacks can require the `site-packages` dependency by generating a [Build
Plan
TOML](https://github.com/buildpacks/spec/blob/master/buildpack.md#build-plan-toml)
file that looks like the following:

```toml
[[requires]]

  # The name of the dependency provided by the Pipenv Install Buildpack is
  # "site-packages". This value is considered part of the public API for the
  # buildpack and will not change without a plan for deprecation.
  name = "site-packages"

  # The Pipenv Install buildpack supports some non-required metadata options.
  [requires.metadata]

    # Setting the build flag to true will ensure that the site-packages
    # dependency is available on the $PYTHONPATH/$PATH for subsequent
    # buildpacks during their build phase. If you are writing a buildpack that
    # needs site-packages during its build process, this flag should be
    # set to true.
    build = true

    # Setting the launch flag to true will ensure that the site-packages
    # dependency is available on the $PYTHONPATH/$PATH for the running
    # application. If you are writing an application that needs site-packages
    # at runtime, this flag should be set to true.
    launch = true
```

## SBOM

This buildpack can generate a Software Bill of Materials (SBOM) for the dependencies of an application.

However, this feature only works if the application already has a
`Pipfile.lock` file. This is due to a limitation in the upstream SBOM
generation library (Syft).

Applications that declare their dependencies via a `Pipfile` but do not include
a `Pipfile.lock` will result in an empty SBOM. Check out the [Paketo SBOM
documentation](https://paketo.io/docs/howto/sbom/) for more information about
how to access the SBOM.

## Usage

To package this buildpack for consumption:
```
$ ./scripts/package.sh --version x.x.x
```
This will create a `buildpackage.cnb` file under the build directory which you
can use to build your app as follows: `pack build <app-name> -p <path-to-app>
-b <cpython buildpack> -b <pipenv buildpack> -b build/buildpackage.cnb -b
<other-buildpacks..>`.

To run the unit and integration tests for this buildpack:
```
$ ./scripts/unit.sh && ./scripts/integration.sh
```
