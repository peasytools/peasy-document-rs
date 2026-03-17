# peasy-document

[![crates.io](https://img.shields.io/crates/v/peasy-document)](https://crates.io/crates/peasy-document)
[![docs.rs](https://docs.rs/peasy-document/badge.svg)](https://docs.rs/peasy-document)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Async Rust client for the [PeasyFormats](https://peasyformats.com) API — convert between Markdown, HTML, CSV, JSON, and YAML document formats. Built with reqwest, serde, and tokio.

Built from [PeasyFormats](https://peasyformats.com), a free online document conversion toolkit with tools for converting, formatting, and validating Markdown, HTML, CSV, JSON, and YAML files.

> **Try the interactive tools at [peasyformats.com](https://peasyformats.com)** — [Document Tools](https://peasyformats.com/), [Document Glossary](https://peasyformats.com/glossary/), [Document Guides](https://peasyformats.com/guides/)

## Install

```toml
[dependencies]
peasy-document = "0.2.0"
tokio = { version = "1", features = ["full"] }
```

Or via cargo:

```bash
cargo add peasy-document
```

## Quick Start

```rust
use peasy_document::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // List available document tools
    let tools = client.list_tools(&Default::default()).await?;
    for tool in &tools.results {
        println!("{}: {}", tool.name, tool.description);
    }

    Ok(())
}
```

## API Client

The client wraps the [PeasyFormats REST API](https://peasyformats.com/developers/) with strongly-typed Rust structs using serde deserialization.

```rust
use peasy_document::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    // Or with a custom base URL:
    // let client = Client::with_base_url("https://custom.example.com");

    // List tools with filters
    let opts = peasy_document::ListOptions {
        page: Some(1),
        limit: Some(10),
        search: Some("markdown".into()),
        ..Default::default()
    };
    let tools = client.list_tools(&opts).await?;

    // Get a specific tool
    let tool = client.get_tool("markdown-to-html").await?;
    println!("{}: {}", tool.name, tool.description);

    // Search across all content
    let results = client.search("markdown", Some(20)).await?;
    println!("Found {} tools", results.results.tools.len());

    // Browse the glossary
    let glossary = client.list_glossary(&peasy_document::ListOptions {
        search: Some("csv".into()),
        ..Default::default()
    }).await?;
    for term in &glossary.results {
        println!("{}: {}", term.term, term.definition);
    }

    // Discover guides
    let guides = client.list_guides(&peasy_document::ListGuidesOptions {
        category: Some("document".into()),
        ..Default::default()
    }).await?;
    for guide in &guides.results {
        println!("{} ({})", guide.title, guide.audience_level);
    }

    // List format conversions
    let conversions = client.list_conversions(&peasy_document::ListConversionsOptions {
        source: Some("json".into()),
        ..Default::default()
    }).await?;

    Ok(())
}
```

### Available Methods

| Method | Description |
|--------|-------------|
| `list_tools(&opts)` | List tools (paginated, filterable) |
| `get_tool(slug)` | Get tool by slug |
| `list_categories(&opts)` | List tool categories |
| `list_formats(&opts)` | List file formats |
| `get_format(slug)` | Get format by slug |
| `list_conversions(&opts)` | List format conversions |
| `list_glossary(&opts)` | List glossary terms |
| `get_glossary_term(slug)` | Get glossary term |
| `list_guides(&opts)` | List guides |
| `get_guide(slug)` | Get guide by slug |
| `list_use_cases(&opts)` | List use cases |
| `search(query, limit)` | Search across all content |
| `list_sites()` | List Peasy sites |
| `openapi_spec()` | Get OpenAPI specification |

Full API documentation at [peasyformats.com/developers/](https://peasyformats.com/developers/).
OpenAPI 3.1.0 spec: [peasyformats.com/api/openapi.json](https://peasyformats.com/api/openapi.json).

## Learn More

- **Tools**: [Markdown to HTML](https://peasyformats.com/tools/markdown-to-html/) · [JSON Formatter](https://peasyformats.com/tools/json-formatter/) · [All Tools](https://peasyformats.com/)
- **Guides**: [How to Convert Markdown](https://peasyformats.com/guides/convert-markdown/) · [All Guides](https://peasyformats.com/guides/)
- **Glossary**: [What is CSV?](https://peasyformats.com/glossary/csv/) · [All Terms](https://peasyformats.com/glossary/)
- **Formats**: [JSON](https://peasyformats.com/formats/json/) · [All Formats](https://peasyformats.com/formats/)
- **API**: [REST API Docs](https://peasyformats.com/developers/) · [OpenAPI Spec](https://peasyformats.com/api/openapi.json)

## Also Available

| Language | Package | Install |
|----------|---------|---------|
| **Python** | [peasy-document](https://pypi.org/project/peasy-document/) | `pip install "peasy-document[all]"` |
| **TypeScript** | [peasy-document](https://www.npmjs.com/package/peasy-document) | `npm install peasy-document` |
| **Go** | [peasy-document-go](https://pkg.go.dev/github.com/peasytools/peasy-document-go) | `go get github.com/peasytools/peasy-document-go` |
| **Ruby** | [peasy-document](https://rubygems.org/gems/peasy-document) | `gem install peasy-document` |

## Peasy Developer Tools

Part of the [Peasy Tools](https://peasytools.com) open-source developer ecosystem.

| Package | PyPI | npm | crates.io | Description |
|---------|------|-----|-----------|-------------|
| peasy-pdf | [PyPI](https://pypi.org/project/peasy-pdf/) | [npm](https://www.npmjs.com/package/peasy-pdf) | [crate](https://crates.io/crates/peasy-pdf) | PDF merge, split, rotate, compress — [peasypdf.com](https://peasypdf.com) |
| peasy-image | [PyPI](https://pypi.org/project/peasy-image/) | [npm](https://www.npmjs.com/package/peasy-image) | [crate](https://crates.io/crates/peasy-image) | Image resize, crop, convert, compress — [peasyimage.com](https://peasyimage.com) |
| peasy-audio | [PyPI](https://pypi.org/project/peasy-audio/) | [npm](https://www.npmjs.com/package/peasy-audio) | [crate](https://crates.io/crates/peasy-audio) | Audio trim, merge, convert, normalize — [peasyaudio.com](https://peasyaudio.com) |
| peasy-video | [PyPI](https://pypi.org/project/peasy-video/) | [npm](https://www.npmjs.com/package/peasy-video) | [crate](https://crates.io/crates/peasy-video) | Video trim, resize, thumbnails, GIF — [peasyvideo.com](https://peasyvideo.com) |
| peasy-css | [PyPI](https://pypi.org/project/peasy-css/) | [npm](https://www.npmjs.com/package/peasy-css) | [crate](https://crates.io/crates/peasy-css) | CSS minify, format, analyze — [peasycss.com](https://peasycss.com) |
| peasy-compress | [PyPI](https://pypi.org/project/peasy-compress/) | [npm](https://www.npmjs.com/package/peasy-compress) | [crate](https://crates.io/crates/peasy-compress) | ZIP, TAR, gzip compression — [peasytools.com](https://peasytools.com) |
| **peasy-document** | [PyPI](https://pypi.org/project/peasy-document/) | [npm](https://www.npmjs.com/package/peasy-document) | [crate](https://crates.io/crates/peasy-document) | Markdown, HTML, CSV, JSON conversion — [peasyformats.com](https://peasyformats.com) |
| peasytext | [PyPI](https://pypi.org/project/peasytext/) | [npm](https://www.npmjs.com/package/peasytext) | [crate](https://crates.io/crates/peasytext) | Text case conversion, slugify, word count — [peasytext.com](https://peasytext.com) |

## License

MIT
