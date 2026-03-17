# peasy-document

[![crates.io](https://img.shields.io/crates/v/peasy-document)](https://crates.io/crates/peasy-document)
[![docs.rs](https://docs.rs/peasy-document/badge.svg)](https://docs.rs/peasy-document)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![crates.io](https://agentgif.com/badge/crates/peasy-document/version.svg)](https://crates.io/crates/peasy-document)
[![GitHub stars](https://agentgif.com/badge/github/peasytools/peasy-document-rs/stars.svg)](https://github.com/peasytools/peasy-document-rs)

Async Rust client for the [PeasyFormats](https://peasyformats.com) API -- convert between Markdown, JSON, YAML, CSV, and other document formats with tools for format identification, MIME type lookup, and structured data transformation. Built with reqwest, serde, and tokio.

Built from [PeasyFormats](https://peasyformats.com), a comprehensive document conversion toolkit offering free online tools for transforming between structured data formats. The glossary covers document formats from lightweight Markdown to structured JSON and YAML, while guides explain format conversion strategies and encoding best practices.

> **Try the interactive tools at [peasyformats.com](https://peasyformats.com)** -- [Markdown to HTML](https://peasyformats.com/doc/markdown-to-html/), [YAML/JSON Converter](https://peasyformats.com/doc/yaml-json-converter/), [Format Identifier](https://peasyformats.com/doc/format-identifier/), and more.

<p align="center">
  <img src="demo.gif" alt="peasy-document demo -- Markdown, JSON, YAML, and CSV conversion tools in Rust terminal" width="800">
</p>

## Table of Contents

- [Install](#install)
- [Quick Start](#quick-start)
- [What You Can Do](#what-you-can-do)
  - [Document Conversion Tools](#document-conversion-tools)
  - [Browse Document Format Reference](#browse-document-format-reference)
  - [Search and Discovery](#search-and-discovery)
- [API Client](#api-client)
  - [Available Methods](#available-methods)
- [Learn More About Document Formats](#learn-more-about-document-formats)
- [Also Available](#also-available)
- [Peasy Developer Tools](#peasy-developer-tools)
- [License](#license)

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

## What You Can Do

### Document Conversion Tools

Document format conversion is a core workflow for developers, data engineers, and content authors. Converting Markdown to HTML powers static site generators and documentation systems. Transforming JSON to YAML (and vice versa) is essential for configuration management across Kubernetes, Docker Compose, and CI/CD pipelines. CSV-to-JSON conversion bridges the gap between spreadsheet data and web APIs.

| Tool | Description | Use Case |
|------|-------------|----------|
| Markdown to HTML | Convert Markdown syntax to semantic HTML | Static sites, documentation pipelines |
| YAML/JSON Converter | Bidirectional YAML and JSON transformation | Kubernetes configs, API payloads |
| Format Identifier | Detect file format from content or extension | File upload validation, data pipelines |

```rust
use peasy_document::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();

    // Fetch the Markdown to HTML conversion tool
    let tool = client.get_tool("markdown-to-html").await?;
    println!("Tool: {}", tool.name);           // Markdown to HTML converter
    println!("Category: {}", tool.category);   // Document conversion category

    // List supported document formats and their MIME types
    let formats = client.list_formats(&Default::default()).await?;
    for fmt in &formats.results {
        println!(".{} -- {} ({})", fmt.extension, fmt.name, fmt.mime_type);
    }

    // List available format conversions from JSON
    let conversions = client.list_conversions(&peasy_document::ListConversionsOptions {
        source: Some("json".into()),
        ..Default::default()
    }).await?;
    println!("Found {} conversion paths from JSON", conversions.results.len());

    Ok(())
}
```

Learn more: [Markdown to HTML Tool](https://peasyformats.com/doc/markdown-to-html/) · [YAML/JSON Converter](https://peasyformats.com/doc/yaml-json-converter/) · [File Format Conversion Guide](https://peasyformats.com/guides/file-format-conversion-guide/)

### Browse Document Format Reference

The document format glossary provides clear definitions for structured data formats, markup languages, and serialization standards. Understanding the differences between JSON and YAML syntax, when CSV is preferable to JSON for tabular data, and how Markdown flavors (CommonMark, GFM) differ helps developers choose the right format for each use case.

| Glossary Term | Description |
|---------------|-------------|
| Markdown | Lightweight markup language for creating formatted text with plain-text syntax |
| CSV | Comma-separated values format for tabular data interchange |
| JSON | JavaScript Object Notation for structured data serialization |
| YAML | Human-readable data serialization format used in configuration files |

```rust
// Browse document format glossary terms
let glossary = client.list_glossary(&peasy_document::ListOptions {
    search: Some("csv".into()),
    ..Default::default()
}).await?;
for term in &glossary.results {
    println!("{}: {}", term.term, term.definition); // Document format definition
}

// Read in-depth guides on format conversion strategies
let guides = client.list_guides(&peasy_document::ListGuidesOptions {
    category: Some("document".into()),
    ..Default::default()
}).await?;
for guide in &guides.results {
    println!("{} ({})", guide.title, guide.audience_level); // Guide title and difficulty
}
```

Learn more: [Markdown Glossary](https://peasyformats.com/glossary/markdown/) · [JSON Glossary](https://peasyformats.com/glossary/json/) · [How to Convert Markdown to Other Formats](https://peasyformats.com/guides/how-to-convert-markdown-to-other-formats/)

### Search and Discovery

The unified search endpoint queries across all document tools, glossary terms, guides, and supported file formats simultaneously. This is useful for building editor plugins, documentation search, or data pipeline tools that need to discover conversion capabilities.

```rust
// Search across all document tools, glossary, and guides
let results = client.search("markdown", Some(20)).await?;
println!("Found {} tools, {} glossary terms",
    results.results.tools.len(),
    results.results.glossary.len()); // Cross-content document format search results
```

Learn more: [CSV Glossary](https://peasyformats.com/glossary/csv/) · [YAML Glossary](https://peasyformats.com/glossary/yaml/) · [All Document Guides](https://peasyformats.com/guides/)

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

## Learn More About Document Formats

- **Tools**: [Markdown to HTML](https://peasyformats.com/doc/markdown-to-html/) · [YAML/JSON Converter](https://peasyformats.com/doc/yaml-json-converter/) · [Format Identifier](https://peasyformats.com/doc/format-identifier/) · [All Tools](https://peasyformats.com/)
- **Guides**: [File Format Conversion Guide](https://peasyformats.com/guides/file-format-conversion-guide/) · [How to Convert Markdown to Other Formats](https://peasyformats.com/guides/how-to-convert-markdown-to-other-formats/) · [All Guides](https://peasyformats.com/guides/)
- **Glossary**: [Markdown](https://peasyformats.com/glossary/markdown/) · [CSV](https://peasyformats.com/glossary/csv/) · [JSON](https://peasyformats.com/glossary/json/) · [YAML](https://peasyformats.com/glossary/yaml/) · [All Terms](https://peasyformats.com/glossary/)
- **Formats**: [All Formats](https://peasyformats.com/formats/)
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
| peasy-pdf | [PyPI](https://pypi.org/project/peasy-pdf/) | [npm](https://www.npmjs.com/package/peasy-pdf) | [crate](https://crates.io/crates/peasy-pdf) | PDF merge, split, rotate, compress -- [peasypdf.com](https://peasypdf.com) |
| peasy-image | [PyPI](https://pypi.org/project/peasy-image/) | [npm](https://www.npmjs.com/package/peasy-image) | [crate](https://crates.io/crates/peasy-image) | Image resize, crop, convert, compress -- [peasyimage.com](https://peasyimage.com) |
| peasy-audio | [PyPI](https://pypi.org/project/peasy-audio/) | [npm](https://www.npmjs.com/package/peasy-audio) | [crate](https://crates.io/crates/peasy-audio) | Audio trim, merge, convert, normalize -- [peasyaudio.com](https://peasyaudio.com) |
| peasy-video | [PyPI](https://pypi.org/project/peasy-video/) | [npm](https://www.npmjs.com/package/peasy-video) | [crate](https://crates.io/crates/peasy-video) | Video trim, resize, thumbnails, GIF -- [peasyvideo.com](https://peasyvideo.com) |
| peasy-css | [PyPI](https://pypi.org/project/peasy-css/) | [npm](https://www.npmjs.com/package/peasy-css) | [crate](https://crates.io/crates/peasy-css) | CSS minify, format, analyze -- [peasycss.com](https://peasycss.com) |
| peasy-compress | [PyPI](https://pypi.org/project/peasy-compress/) | [npm](https://www.npmjs.com/package/peasy-compress) | [crate](https://crates.io/crates/peasy-compress) | ZIP, TAR, gzip compression -- [peasytools.com](https://peasytools.com) |
| **peasy-document** | [PyPI](https://pypi.org/project/peasy-document/) | [npm](https://www.npmjs.com/package/peasy-document) | [crate](https://crates.io/crates/peasy-document) | **Markdown, HTML, CSV, JSON conversion -- [peasyformats.com](https://peasyformats.com)** |
| peasytext | [PyPI](https://pypi.org/project/peasytext/) | [npm](https://www.npmjs.com/package/peasytext) | [crate](https://crates.io/crates/peasytext) | Text case conversion, slugify, word count -- [peasytext.com](https://peasytext.com) |

## License

MIT
