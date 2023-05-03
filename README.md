# Content Source Maps Specification

# Background

When working with composable systems and structured content, the information end users see originates from multiple sources: multiple “parts” that content creators author, that are assembled together seamlessly into a “whole” to be presented to users. Having access to metadata of the specific “parts”, but directly from the “whole” - the assembled end-user experience - it’s extremely useful for content creators, reviewers, and developers, for example, telling where each individual fragment of content came from, who edited last and when it was last updated.

Content Source Maps is a standard representation to annotate fragments in a JSON document with metadata about its origin: the field, document, and dataset it originated from. We do this with a separate document alongside the content that provides the metadata without changing the layout of the original document.

Today Content Source Maps enables annotating JSON documents with “source” metadata, allowing end users to navigate directly to the source to edit it. In the future, content source maps will also enable annotating JSON documents with arbitrary metadata for other use cases.

# **Document revisions**

| Tues 25. April | Michael Wain | Initial Revision |
| -------------- | ------------ | ---------------- |

# Terminology

| Term                 | Definition                                                                                                                                                                                 |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Content              | The information displayed to end users within a JSON document                                                                                                                              |
| JSON Value           | A value within a JSON document, such as a string, number, object, array, or boolean                                                                                                        |
| Mapping              | A connection between a content value and its source or sources                                                                                                                             |
| Source               | The origin of the content, such as a JSON document and path                                                                                                                                |
| Normalised JSON Path | A string representing the location of a value within a JSON document in a standardised format. See https://datatracker.ietf.org/doc/html/draft-ietf-jsonpath-base-13#name-normalized-paths |

# Overview of Content Source Map

The Content Source Map offers a standard method for representing the mapping between content values and their sources.

Example Content Source Map:

```json
{
  "documents": [
    {
      "_id": "author-1"
    },
    {
      "_id": "author-2"
    }
  ],
  "paths": ["$['name']"],
  "mappings": {
    "$[0]": {
      "type": "value",
      "source": {
        "type": "documentValue",
        "document": 0,
        "path": 0
      }
    },
    "$[1]": {
      "type": "value",
      "source": {
        "type": "documentValue",
        "document": 1,
        "path": 0
      }
    }
  }
}
```

## Mapping

Mappings is a Map, where the key is a Normalised JSON Path representing the location of the content value within the JSON document, and the value is the mapping that connects the content value to its source or sources.

## Source

Source describes the origin of the content. It generally represents a JSON Document and the Normalised JSON Path inside the document where the content originated.

## Lookup Tables

The `documents` and `paths` properties within the Content Source Map serve as lookup tables to reduce the overall size of the map.

## Example

Imagine these three independent JSON documents exists:

A document representing the author “George Orwell”:

```json
{
  "_id": "author-george-orwell-4c9f",
  "_type": "author",
  "died": "1950-01-21",
  "dob": "1903-05-25",
  "firstName": "George",
  "lastName": "Orwell"
}
```

Another document representing the book “Animal Farm” by author George Orwell (a reference to the first document)

```json
{
  "_id": "book-animal-farm-3856",
  "_type": "book",
  "description": "It tells the story of a group of farm animals who rebel against their human farmer",
  "title": "Animal Farm",
  "author": {
    "_ref": "author-george-orwell-4c9f"
  }
}
```

And another document representing the book “Nineteen Eighty-Four” by author George Orwell as well (a reference to the first document as well)

```json
{
  "_id": "book-1984-12eb",
  "_type": "book",
  "description": "Nineteen Eighty-Four (also published as 1984) is a dystopian social science fiction novel and cautionary tale by English writer George Orwell.",
  "title": "Nineteen Eighty-Four",
  "author": {
    "_ref": "author-george-orwell-4c9f"
  }
}
```

If these three documents are composed into the following document:

```json
[
  {
    "authorName": "Orwell",
    "booksWritten": ["Nineteen Eighty-Four", "Animal Farm"]
  }
]
```

A content source map for this composed document will look like this:

```json
{
  "documents": [
    {
      "_id": "author-george-orwell-4c9f"
    },
    {
      "_id": "book-1984-12eb"
    },
    {
      "_id": "book-animal-farm-3856"
    }
  ],
  "paths": [
    "$['lastName']",
    "$['title']"
  ],
  "mappings": {
    "$[0]['authorName']": {
      "source": {
        "document": 0,
        "path": 0,
        "type": "documentValue"
      },
      "type": "value"
    },
    "$[0]['booksWritten'][0]": {
      "source": {
        "document": 1,
        "path": 1,
        "type": "documentValue"
      },
      "type": "value"
    },
    "$[0]['booksWritten'][1]": {
      "source": {
        "document": 2,
        "path": 1,
        "type": "documentValue"
      },
      "type": "value"
    }
  }
}
```

Observe that the content source map includes:

- Under `documents`, a list of documents from where the content in the composed document comes from, i.e.:

  - author document with id `author-george-orwell-4c9f`
  - book document with id `book-1984-12eb`
  - and book document with id `book-animal-farm-3856`

  Observe in this example, the “\_id” attribute is used to identify referenced documents - in a content source map, any arbitrary attribute can be used to identify content sources.

- Under `paths`, a list of the attribute names from where the content in the composed document comes from, i.e.:

  - attribute name `lastName`(specified as `"$['lastName']"`)
  - and attribute name `title` (specified as `"$['title']"`)

  - the first map entry…

  ```json
  "$[0]['authorName']": {
    "source": {
      "document": 0,
      "path": 0,
      "type": "documentValue"
    },
    "type": "value"
  }
  ```

    … describes that, in the first element in the composed document (`$[0]`), the attribute “authorName” (`['authorName']`) comes from (`source:`) the document in the first position of the “documents” set (`"document": 0`, which is `author-george-orwell-4c9f` ), and the attribute in the first position in the “paths” set (`"path": 0`, which is `lastName`);
    i.e. the value `"authorName": "Orwell"` in the response, comes from the document `author-george-orwell-4c9f`, and attribute `lastName`.

  - the second map entry…

  ```json
  "$[0]['booksWritten'][0]": {
    "source": {
      "document": 1,
      "path": 1,
      "type": "documentValue"
    },
    "type": "value"
  }
  ```

    … describes that, in the first element in the composed document (`$[0]`), the attribute “booksWritten” (`['booksWritten']`), it’s first element (`[0]`) comes from (`source:`) the document in the second position of the “documents” set (`"document": 1`, which is `book-1984-12eb` ), and the attribute in the second position in the “paths” set (`"path": 1`, which is `title`);
    i.e. the value in the first element of `booksWritten` in the response (the string `"Nineteen Eighty-Four"`), comes from the document `book-1984-12eb`, and attribute `title`.

  - the third map entry…

  ```json
  "$[0]['booksWritten'][1]": {
    "source": {
      "document": 2,
      "path": 1,
      "type": "documentValue"
    },
    "type": "value"
  }
  ```

    … describes that, in the first element in the composed document (`$[0]`), the attribute “booksWritten” (`['booksWritten']`), it’s second element (`[1]`) comes from (`source:`) the document in the third position of the “documents” set (`"document": 2`, which is `book-animal-farm-3856`), and the attribute in the second position in the “paths” set (`"path": 1`, which is `title`);
    i.e. the value in the second element of `booksWritten` in the response (the string `"Animal Farm"`), comes from the document `book-animal-farm-3856`, and attribute `title`.

# Content Source Map Format

```tsx
type Source = DocumentValueSource | LiteralSource | UnknownSource;
type Mapping = ValueMapping | RangeMapping | DerivedMapping;

type Document = any;

type ContentSourceMapping = {
  mappings: Record<string, Mapping>;
  documents: Array<Document>;
  paths: Array<string>;
};

type DocumentValueSource = {
  type: 'documentValue';
  document: number;
  path: number;
};

type LiteralSource = {
  type: 'literal';
};

type UnknownSource = {
  type: 'unknown';
};

type ValueMapping = {
  type: 'value';
  source: Source;
};

type RangeMapping = {
  type: 'range';
  ranges: Array<{
    start: number;
    end: number;
    source: Source;
  }>;
};

type DerivedMapping = {
  type: 'derived';
  sources: Array<Source>;
};
```

## Mapping Format

The Content Source Map's mapping format is designed to accommodate various content scenarios, including single values originating from a single source or derived values from multiple sources.

### Single Value Mapping

In situations where content is derived from a single source, the below mapping format applies.

```tsx
type ValueMapping = {
  type: 'value';
  source: Source;
};
```

This type of mapping is used for content that has a singular origin.

### Derived Value Mapping

For content that is derived from multiple sources, the below mapping format applies.

```tsx
type DerivedMapping = {
  type: 'derived';
  sources: Array<Source>;
};
```

This type of mapping is used for content that has complex origin, such as values generated through calculations, concatenations or transformations involving multiple source values.

### Range Value Mapping

In cases where content is composed of multiple source values with known positions within the resulting value, the below mapping format applies:

```tsx
type RangeMapping = {
  type: 'range';
  ranges: Array<{
    start: number;
    end: number;
    source: Source;
  }>;
};
```

This type of mapping is especially relevant for content that has been combined from several sources, such as concatenated fields.

## Source Format

The Source is a vital component of the Content Source Map mapping format, as it conveys the origin of content, enabling users to trace it back to its source.

### Document Value

The Document Value source represents a value that originates from a single JSON document and a JSON Path that precisely indicates the location of the value within the document.

```tsx
type DocumentValueSource = {
  type: 'documentValue';
  document: number;
  path: number;
};
```

### Literal

The Literal source represents content values that are not associated with any specific source. Instead, these values are literal values provided directly by the user. This source type is useful when dealing with static or user-defined content.

```tsx
type LiteralSource = {
  type: 'literal';
};
```

### Unknown

In certain situations, it may not be possible to determine the origin of a content value, or the information about its origin may have been lost. In these cases, the Unknown source type can be used to indicate the untraceable nature of the content value.

```tsx
type UnknownSource = {
  type: 'unknown';
};
```

# Resolving a Source

<aside>
🚧 In a new revision we will open source a library to resolve sources
</aside>

To resolve a content source, follow these steps:

1. **Construct the JSON Path**: Determine the full Normalised JSON Path of the content value within a JSON Document
2. **Look up the mapping:** Use the Normalised JSON Path to find the corresponding mapping with the Content Source Map object, which contains the relationships between content values and their sources.
3. **Find the closest string prefix:** If an exact match is not found, identify the closest string prefix that matches the result Normalised JSON Path. This method locates the most specific mapping that aligns with the given path.
4. **Append the path suffix**: If a matching mapping is found, append any remaining path suffix to the source path. This step ensures that the final source path is an accurate representation of the values location in the source document.

By following this process, you can efficiently resolve a mapping for any content value.
