# How to Develop TensorBoard

TensorBoard at HEAD relies on the nightly installation of TensorFlow: this allows plugin authors to use the latest features of TensorFlow, but it means release versions of TensorFlow may not suffice for development. We recommend installing TensorFlow nightly in a [Python virtualenv](https://virtualenv.pypa.io), and then running your modified development copy of TensorBoard within that virtualenv. To install TensorFlow nightly within the virtualenv, as well as TensorBoard's runtime and tooling dependencies, you can run:

```sh
$ virtualenv tf
$ source tf/bin/activate
(tf)$ pip install --upgrade pip
(tf)$ pip install tf-nightly -r tensorboard/pip_package/requirements.txt -r tensorboard/pip_package/requirements_dev.txt
```

TensorBoard builds are done with [Bazel](https://bazel.build), so you may need to [install Bazel](https://docs.bazel.build/versions/master/install.html). The Bazel build will automatically "vulcanize" all the HTML files and generate a "binary" launcher script. When HTML is vulcanized, it means all the script tags and HTML imports are inlined into one big HTML file. Then the Bazel build puts that index.html file inside a static assets zip. The python HTTP server then reads static assets from that zip while serving.

You can build and run TensorBoard via Bazel (from within the TensorFlow nightly virtualenv) as follows:

```sh
(tf)$ bazel run //tensorboard -- --logdir /path/to/logs
```

You may see warnings about “Limited tf.compat.v2.summary API due to missing TensorBoard installation” appear when you run TensorBoard. These are spurious: you can ignore them. (See [an explanation of why these warnings occur][why-warnings] if you’re curious.)

[why-warnings]: https://github.com/tensorflow/tensorboard/issues/2968#issuecomment-558405994

For any changes to the frontend, you’ll need to install [Yarn][yarn] to lint your code (`yarn lint`, `yarn fix-lint`). You’ll also need Yarn to add or remove any NPM dependencies.

For any changes to the backend, you’ll need to install [Black][black] to lint your code (run `black .`). Our `black` version is specified in `requirements_dev.txt` in this repository. Black only runs on Python 3.6 or higher, so you may want to install it into a separate virtual environment and use a [wrapper script to invoke it from any environment][black-wrapper].

You may wish to configure your editor to automatically run Prettier and Black on save.

To generate fake log data for a plugin, run its demo script. For instance, this command generates fake scalar data in `/tmp/scalars_demo`:

```sh
(tf)$ bazel run //tensorboard/plugins/scalar:scalars_demo
```

If you have Bazel≥0.16 and want to build any commit of TensorBoard prior to 2018-08-07, then you must first cherry-pick [pull request #1334][pr-1334] onto your working tree:

```
$ git cherry-pick bc4e7a6e5517daf918433a8f5983fc6bd239358f
```

[black]: https://github.com/psf/black
[black-wrapper]: https://gist.github.com/wchargin/d65820919f363d33545159138c86ce31
[pr-1334]: https://github.com/tensorflow/tensorboard/pull/1334
[yarn]: https://yarnpkg.com/

## Pro tips

You may find the following optional tips useful for development.

### Ignoring large cleanup commits in `git blame`

```shell
git config blame.ignoreRevsFile .git-blame-ignore-revs  # requires Git >= 2.23
```

We maintain a list of commits with large diffs that are known to not have any
semantic effect, like mass code reformattings. As of Git 2.23, you can configure
Git to ignore these commits in the output of `git blame`, so that lines are
blamed to the most recent “real” change. Set the `blame.ignoreRevsFile` Git
config option to `.git-blame-ignore-revs` to enable this by default, or pass
`--ignore-revs-file .git-blame-ignore-revs` to enable it for a single command.
When enabled by default, this also works with editor plugins like
[vim-fugitive]. See `git help blame` and `git help config` for more details.

[vim-fugitive]: https://github.com/tpope/vim-fugitive
