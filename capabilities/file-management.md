# File Management

## Generation Instructions
File Management handles upload, storage, retrieval, and management of files and media assets. If `config.attachment-mode` is `entity`, files attach to specific entity records declared in `config.attaches-to`. If `attachment-mode` is `standalone`, files exist independently in a general library.

Generate an upload UI with drag-and-drop and file picker support, upload progress indication, and client-side validation of file type and size against `config.accepted-types` and `config.max-file-size-mb` before upload begins. Reject disallowed types and oversized files with clear error messages.

Generate storage integration for the provider declared in `config.storage-provider`. Store files with a generated key (do not use user-supplied filenames as storage keys). Generate a file metadata model capturing: id, original filename, mime type, size, storage key, uploader userId, attached entity reference (if entity mode), and created_at.

Generate download and inline preview surfaces. For image types, generate thumbnail previews using the dimensions in `config.image-processing` if image processing is enabled. Thumbnails are generated on upload.

Generate deletion. If the Soft Delete capability is present on the file entity, use soft delete. Otherwise, hard delete from storage and database.

Generate admin surfaces for each enabled entry in `config.admin-surface`.

capability: file-management
type: attachable
version: 1.0

depends-on: []

emits:
  - event: file-management.uploaded
    payload:
      fileId: string
      userId: string
      entityType: string | null
      entityId: string | null
      mimeType: string
      sizeBytes: number
  - event: file-management.deleted
    payload:
      fileId: string
      userId: string

config:
  attachment-mode: entity       # entity | standalone

  attaches-to:
    - entity: example-entity    # replace with actual entity names (entity mode only)

  accepted-types:               # mime type patterns; use * for any
    - image/*
    - application/pdf

  max-file-size-mb: 10

  storage-provider: s3          # s3 | gcs | azure | local

  image-processing:
    enabled: true
    thumbnail-width: 200
    thumbnail-height: 200

  admin-surface:
    file-library: true          # view and search all uploaded files
    bulk-delete: true
