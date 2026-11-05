# python-ajsonrpc

A backport of Ubuntu's own `python-ajsonrpc` to the older series, published to
the
[Forgejo Debian registry](https://forgejo.bobbalisa.org/pillarsdotnet/-/packages)
for **noble** (24.04) and **resolute** (26.04).

`python-ajsonrpc` entered Ubuntu in **stonking** (26.10) and nothing older
carries it, while [`platformio`](https://forgejo.bobbalisa.org/pillarsdotnet/platformio)
and [`pioarduino`](https://forgejo.bobbalisa.org/pillarsdotnet/pioarduino) need
`python3-ajsonrpc` on every series for PIO Home. That gap is the only reason
this repository exists.

## What is in here, and what is not

**Only the CI.** There is no `debian/` directory, no patches and no copy of
the source: the delta against the archive is empty, because a backport that
changes the source is not a backport. The source package comes from
stonking's archive at build time, so a Debian or Ubuntu update to it is
inherited without anyone editing this repository.

```
drops/<series>       # control lines to strip for a series, if one ever needs it
.forgejo/
    workflows/registry.yml   # the matrix, and a weekly cron
    workflows/mirror.yml     # rebuild the github branch, push it to GitHub
    registry-upload          # the whole build
```

The build is:

```
apt-get source python-ajsonrpc   (deb-src for stonking)
  ->  strip any drops/<series> lines from debian/control
  ->  write a changelog entry
  ->  apt-get build-dep ./       (falling back to nocheck if that will not resolve)
  ->  dpkg-buildpackage -b
  ->  upload the debs to the registry
```

It produces `python3-ajsonrpc` and `python-ajsonrpc-doc`, both
`Architecture: all`, but each series is still built in its own container:
`${python3:Depends}` has to come out as that series' `python3` resolves it.

## Versions

```
1.2.0-2~ppa1~ubuntu24.04.1      noble
1.2.0-2~ppa1~ubuntu26.04.1      resolute
```

`<source version>~ppa<N>~ubuntu<release>.1` sorts **below** the archive's own
`1.2.0-2` on purpose: if noble or resolute ever gains the real package, the
archive wins and this backport steps aside. It sorts in archive order across
series, so a host upgrading noble to resolute still moves forward.

Because the version embeds the source version, *already in the registry* is a
complete answer to *is there anything to do*: the weekly cron run rebuilds
nothing until the archive moves. To force a rebuild of an unchanged source
package — after a fix to `registry-upload`, say — raise `PPA_REVISION` in that
script.

## When a series is missing a dependency

A source package taken from a newer suite can want something the target series
has not got. Two escape hatches, in order of preference:

1. **Test dependencies**: nothing to do. When the full build-dep set will not
   resolve, the script retries under `nocheck` and builds without the test
   suite rather than not building at all. The log says so when it happens.
2. **Runtime dependencies**: add `drops/<series>` with one extended regex per
   line, each matching a `debian/control` line to strip for that series — the
   mechanism the retired Launchpad workflow spelled `debian/noble-drop`. A
   regex that matches nothing fails the build, so a control file that moves
   under a drop is noticed rather than silently ignored.

Neither is needed today: `1.2.0-2` builds and installs on noble and resolute
as it stands.

## Related

- [`platformio`](https://forgejo.bobbalisa.org/pillarsdotnet/platformio) and
  [`pioarduino`](https://forgejo.bobbalisa.org/pillarsdotnet/pioarduino) — what
  needs this.
- [`gimp-deb`](https://forgejo.bobbalisa.org/pillarsdotnet/gimp-deb) — the same
  delta-only arrangement, with a patch series rather than an empty one.
