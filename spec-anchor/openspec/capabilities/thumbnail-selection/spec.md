# Thumbnail Selection — Specification

> Version: 1.0 | Status: Implemented | Last updated: 2026-04-11

## Purpose

Allow the user to review QUICKLOOK thumbnails from the CDSE query results and
build a download list by selecting products by index, rather than downloading
everything that passes the query filters.

## Functional Requirements

### REQ-TNSEL-001: Numbered thumbnails
The system SHALL display each qualifying product with a sequential integer
index `[N]` printed above or alongside its QUICKLOOK thumbnail.

### REQ-TNSEL-002: Persistent product list
The system SHALL store all products that pass the cloud-cover and tile-area
filters in a list (`display_products`) ordered by display index.

### REQ-TNSEL-003: Index-based selection
The system SHALL provide a `selected_indices` variable the user edits to
choose which products to download; setting it produces a `download_list`.

### REQ-TNSEL-004: Selection confirmation
The system SHALL print the names of selected products when `selected_indices`
is evaluated, so the user can confirm the list before triggering a download.

## Acceptance Scenarios

### SCENARIO-TNSEL-001: User selects a subset
**GIVEN** a query returns five products  
**WHEN** the user sets `selected_indices = [0, 3]`  
**THEN** `download_list` contains exactly products 0 and 3

### SCENARIO-TNSEL-002: Empty selection skips download
**GIVEN** `selected_indices = []`  
**WHEN** the download cell runs  
**THEN** no authentication is attempted and a message is printed

## Implementation Status (2026-04-11)

**Status**: Implemented

### What's Built
- `show_thumbnail` updated to accept `index` parameter (`CDSE_OData_Workflow_v1.ipynb`, cell `show-thumbnail-cc`)
- Query loop populates `display_products`, prints `[N]` labels (`query-loop-cc`)
- `selected_indices` / `download_list` cell (`sel-indices-cc`)
- Download cell checks for empty list before calling auth (`download-cc`)

### Deviations from Spec
- None

### Deferred
- None
