# Baseline for open-source projects

## Introduction

Before releasing software to the world, we must ensure that it is made available in a way that is useful for others outside of our organization.

There have been many attempts to model the traits of successful, mature software, none of them perfect. This document draws on well-known [non-functional requirements](https://en.wikipedia.org/wiki/Non-functional_requirement) to produce a minimal baseline that projects should meet before they are widely shared with the world.

## When to open-source

Prefer developing and sharing work in the open. If your code can help others, or help make your publications more trustworthy, try to open-source it. Plus, as [Bert Hubert points out](https://berthub.eu/articles/posts/on-long-term-software-development/), "[o]pen source code often has higher standards, and it is a great mechanism of keeping you on track."

It is not necessarily the case that projects must meet this baseline before being open-sourced.

Some projects may be developed in the open from the first line of code; in this case, the only prerequisite is to clearly declare an open-source license for the code before starting development. This will help avoid misunderstandings and disputes later on.

Other projects may be open-sourced long after they were first developed; meeting as many of these requirements as possible beforehand will make it easier for other people to discover, use, and contribute to the project.

In all cases, after it becomes clear that the project will have a long-term future, projects should prioritize meeting this baseline as soon as possible. Delaying will make it more difficult to meet the baseline as the project grows.

## Licensing

All source code or creative work should be assigned a license before it is made public.

Very small excerpts of code or text, trivial enough to be commonly understood as freely available under [United States fair use standards](https://en.wikipedia.org/wiki/Fair_use), are exempt from this requirement. However, if in any doubt, assign a license to avoid the uncertainty this causes; see [the many debates on this issue at Stack Overflow](https://duckduckgo.com/?q=stack%20overflow%20code%20license&t=ffab&ia=web).

Our projects use the [3-clause BSD license](https://opensource.org/license/BSD-3-clause) for code and the [Creative Commons Attribution license (aka CC-BY)](https://creativecommons.org/licenses/by/4.0/) (latest version available) for non-code content such as text and images. To assign a license, add a `LICENSE.md` file to your Git repository. Modify and use [the text and format from one of our existing projects, such as PnPQ](https://github.com/moonshot-nagayama-pj/PnPQ/blob/main/LICENSE.md).

Use of other licenses must be justified and discussed with the team; for example, if a project requires linking to or incorporating GPL-licensed code and there is no substitute available, we may have to release our own project under the GPL or find a way to structure the project such that the terms of the GPL are not violated.

Even when copying or linking code from other BSD or BSD-compatible projects, keep in mind that attribution is almost always required. For example, if you copy a file from an Apache-licensed open-source project, you must clearly attribute that copied code to the original project and provide a full copy of the Apache license with your source code. Just as we expect attribution for our own code, it is important that we reciprocate and honor the expectations of other open-source developers; [this is a good example of how failing to do so harms people who have invested significant effort into open-source projects](https://philiplaine.com/posts/getting-forked-by-microsoft/).

We do not require a [contributor license agreement (CLA)](https://en.wikipedia.org/wiki/Contributor_License_Agreement) or [developer certificate of origin (DCO)](https://en.wikipedia.org/wiki/Developer_Certificate_of_Origin). Copyright remains with the original contributors.

The most appropriate licensing for AI-related training data and models is an open question as of 2025; if we should produce such a project, we should discuss our approach before open-sourcing.

## Documentation

* The README should contain information on how to get started using the project, how to contact us and file issues, how to contribute, and how to access further documentation.
* The public API should have reference documentation. [KMI](https://qmi.readthedocs.io/en/latest/) ([GitHub](https://github.com/QuTech-Delft/QMI)) is a good example of a project similar to PnPQ.
    - Python projects should use Sphinx and ReStructuredText. API documentation should be written inline with the code. The generated documentation should be hosted on GitHub Pages; see PnPQ for an example of how to automate this.
    - Rust projects should use rustdoc and host on [docs.rs](https://docs.rs/about).
* Sufficient explanation and tutorial documentation should be present such that the project is usable without asking one of the project authors for help.
* Consider the [Diátaxis](https://diataxis.fr) framework when writing documentation.
* We have yet to adopt a comprehensive style guide, but keep the following general guidelines in mind:
    - Use American English spelling.
    - Use semantic markup whenever possible. For example, if you are marking sections in a document, use `<h2>`, `<h3>`, etc. tags instead of bold or italic. Similarly, when referencing a method or class, try to link to the method or class in question the first time it is mentioned in a section, rather than simply using fixed-width text.

## Logging

### Python

* All log statements should use [structlog](https://www.structlog.org/en/stable/).
    - structlog should be [configured to integrate with standard-library logging](https://www.structlog.org/en/stable/standard-library.html) so that log output from third-party libraries also appears in our logs.
* All log statements at the `INFO` level and above should use a machine-parseable event ID ([examples in PnPQ](https://github.com/moonshot-nagayama-pj/PnPQ/blob/main/src/pnpq/events.py)) that is unique within the scope of the program. The meaning of these IDs should be documented. Doing so makes logs much easier to use in aggregated monitoring systems.
    -  `DEBUG` log messages may use event IDs.
    - Structured logging practices should be followed when possible. For example, if a log message includes a numerical value that could be meaningful to a computer, it should be logged as a key-value pair so that computers can easily parse it.
* At the `INFO` level and above, stub implementations should emit the same events as their real counterparts.

### Log levels

Log levels should be used similarly across programming languages. This helps make writing filters and alerts easier. We only use the three most common log levels: `DEBUG`, `INFO`, and `ERROR`.

* `TRACE`: Not present in Python stdlib logging; do not use.
* **`DEBUG`**: Trace program execution to help identify the cause of problems. May be very high-volume and verbose. For performance reasons, should be turned off when the software is deployed in production.
* **`INFO`**: Report normal operations. The volume and size of messages should be small.
* `WARNING`: Avoid using this level. Instead, use `ERROR`. Often, log aggregators may filter and alert on `ERROR` but not `WARNING`, meaning that messages logged at this level that should be acted on are instead lost.
* **`ERROR`**: Report application errors. The application may or may not continue execution after this error. Stack traces and exception objects should be included when available.
* `CRITICAL`: Not present in Rust's [log crate](https://docs.rs/log/latest/log/); do not use.

## Builds and code quality

### Python

Use [`uv`](https://docs.astral.sh/uv/) to manage builds.

Write type-safe Python code. Use [mypy](https://mypy.readthedocs.io/en/stable/) for static type checking. If you are using [other type checkers](https://github.com/emmatyping/python-typecheckers), ensure that your code also passes mypy checks.

Automatically run static analysis and unit tests on each pull request. The same checks should be runnable locally. An implementation can be found in PnPQ's [`bin/check.bash`](https://github.com/moonshot-nagayama-pj/PnPQ/blob/main/bin/check.bash). Until we have the time to better standardize and automate across our projects, consider PnPQ's checks as the minimum baseline for static analysis.

### Unit testing

We should strive to write unit tests at the same time we implement new features, and keep code coverage high, at over 80%.

Much of our code interfaces with real hardware. In order to allow unit testing, we should structure our code to isolate the hardware interfaces and make it easy to mock hardware behavior.

## Backwards compatibility

* [semver](https://semver.org/) should be understood and used to version our projects. We should try to avoid making backwards-incompatible changes to our API.

### Python

* The minimum required Python version should be clearly stated and used in testing. The minimum version should only be raised if necessary.
* [Run tests against lowest acceptable dependency versions](https://github.com/moonshot-nagayama-pj/PnPQ/issues/87) in addition to the newest versions of dependencies.

## Availability

* The project should be uploaded to the commonly used package repository for that language.
    - Python projects should be uploaded to PyPI.
* The release and upload process should be triggerable via GitHub Actions. It should not happen automatically on every merge to main. We should intentionally choose to release each new version, and intentionally choose the version number appropriately using [semver](https://semver.org/) guidelines.
* When the project is sufficiently mature, it should be promoted through appropriate means, including Slack groups, conference posters, and word of mouth.

## Citeability

Major projects should be citeable in academic papers. In some cases, there may be an associated academic paper that can be cited; however, following [proposed software citation principles](https://doi.org/10.7717/peerj-cs.86), it is better to directly cite the software itself.

In order to make a project citeable, citation metadata must first be added to the Git repository using the [Citation File Format (CFF)](https://citation-file-format.github.io/). Then, a [digital object identifier (DOI)](https://en.wikipedia.org/wiki/Digital_object_identifier) must be generated using [Zenodo](https://zenodo.org/) to uniquely and reliably identify the project. Finally, this DOI must be added to the CFF file.

* Add the `CITATION.cff` file to the repository.
    - [The PR adding `CITATION.cff` to PnPQ](https://github.com/moonshot-nagayama-pj/PnPQ/pull/154) is a good example. At this point, the CFF file has no DOI and no version.
    - Pay attention to [the list of CFF fields that Zenodo supports](https://help.zenodo.org/docs/github/describe-software/citation-file/).
    - Be sure to validate the file using [`cffconvert`](https://github.com/citation-file-format/cffconvert). If this is a Python project, it is trivial to add `cffconvert` to the project's dev dependencies and run this command in `check.bash`.
    - Try to make sure that the metadata in this file matches that in `Cargo.toml`, `pyproject.toml`, and the README.
* [Link Zenodo to your GitHub repository](https://help.zenodo.org/docs/github/enable-repository/).
* Create a GitHub release. Zenodo will automatically archive the release and create two DOIs: [a concept DOI and a version DOI](https://zenodo.org/help/versioning). The concept DOI points to the software project as a whole; the version DOI points to the files uploaded for a specific version of the software.
* Obtain the concept DOI from Zenodo and add it to `CITATION.cff`. At the same time, update the project README to include information on how to cite the project.
    - Again, be sure to validate the file using `cffconvert`.
    - [The text in the PnPQ README can be re-used for new projects](https://github.com/moonshot-nagayama-pj/PnPQ/tree/main?tab=readme-ov-file#citing).

At this point, it should be possible to cite the project. Zenodo will automatically create a new DOI for each subsequent GitHub release. Note that there is still no version number in `CITATION.cff`; we may improve automation to include this in the future.

If you created an irregular release to trigger Zenodo the first time, you may wish to delete it and its associated tag. This will not delete the record at Zenodo.

## Further references

* [Wikipedia's OpenSource Maturity Model page](https://en.wikipedia.org/wiki/OpenSource_Maturity_Model) is copied from a defunct organization's model, but touches on many of the same topics.
* The [FINOS Open Source Maturity Model](https://osr.finos.org/docs/bok/osmm/introduction) comes from the Fintech Open Source Foundation.
