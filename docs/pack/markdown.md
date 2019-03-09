The Markdown pack provides parsing support and other utilities for working with
Markdown files. The automation around this docs site uses this pack.

[GitHub][]

[API Doc][api-doc]

[github]: https://github.com/atomist/sdm-pack-markdown (GitHub Repository)
[api-doc]: https://atomist.github.io/sdm-pack-markdown/ (API Docs)

## Parsing

Use the `RemarkFileParser` when evaluating [path expressions](../developer/pxe.md)
to transform Markdown files.

## Code Transforms

[`updateTitle`](https://atomist.github.io/sdm-pack-markdown/modules/_lib_transform_updatetitle_.html#updatetitle) returns
a transform that changes the first top-level header of a Markdown file.

### Update Title

Update the title of a Markdown document. This is useful for changing the title of README.md
in a new project.
