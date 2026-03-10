# Search

## Generation Instructions
Search provides full-text and filtered search across application entities declared in `config.attaches-to`. Generate a search index configuration for each attached entity specifying which fields are indexed. Generate index update triggers on entity create, update, and delete to keep the search index current.

Generate the search API endpoint that accepts a query string and optional filters, queries the configured provider, and returns results. Generate the search UI: a search input component, results list, and faceted filter controls for each entity's declared `facetable-fields`.

If `config.permission-scoped` is true, filter search results to only include records the requesting user is authorized to access. If the Permissions capability is present, apply its permission checks to each result. Records the user cannot access must not appear in results even if they match the query.

Generate provider integration for `config.provider`. For the `database` provider, use the database's native full-text search (PostgreSQL `tsvector`). For external providers (elasticsearch, typesense, meilisearch), generate the index configuration, client setup, and index synchronization logic.

If `config.emit-analytics` is true, emit `search.performed` on each query execution.

capability: search
type: attachable
version: 1.0

depends-on:
  - auth

emits:
  - event: search.performed
    payload:
      userId: string
      query: string
      entityType: string | null   # null if searching across all types
      resultCount: number
    # only emitted if emit-analytics is true

config:
  provider: database            # database | elasticsearch | typesense | meilisearch

  permission-scoped: true       # filter results to records the user can access

  emit-analytics: false         # emit search.performed event on each query

  attaches-to:
    - entity: example-entity
      indexed-fields:
        - name
        - description
      facetable-fields:
        - status
        - category
