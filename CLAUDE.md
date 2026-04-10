# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains an interactive HTML mockup demonstrating the improved UX for Salesforce Data Cloud retriever deletion when dependencies exist. The feature shows all retriever dependencies in an Integrations tab with navigable links, allowing users to identify and remove blocking references before deletion.

## Key Concepts

### Retriever Dependencies
Retrievers in Data Cloud can be referenced by multiple components:
- **Agent Actions**: AI agent functions that invoke retrievers
- **Flows**: Salesforce Flow automations using retrievers
- **Prompt Templates**: AI prompt templates referencing retrievers
- **Ensemble Retrievers**: Parent retrievers containing other retrievers
- **Data Libraries**: Agentforce data libraries using search indexes tied to retrievers
- **Companion Org References**: Cross-org references from Data Cloud One multi-org setups (non-clickable, informational only)

### API Endpoint Structure
The feature relies on `GET /ssot/ml/retrievers/{id}/integrations` which returns:
- Reference ID, name, label, and type
- Navigation URL for clickable links (null for companion org refs)
- Remote reference flag and org owner for cross-org dependencies

### User Flow
1. User attempts to delete a retriever
2. Confirmation modal appears
3. If dependencies exist, error modal directs to Integrations tab
4. Integrations tab shows all dependencies organized by type
5. User clicks through to each dependency to remove references
6. User can then successfully delete the retriever

## File Structure

- `retriever-deletion-mockup.html` - Complete interactive mockup with embedded CSS and JavaScript
- `README.md` - Documentation on using and understanding the mockup

## Mockup Implementation Details

### Styling
- Uses Salesforce Lightning Design System (SLDS) CDN for base styles
- Custom CSS for layout, modals, and component-specific styling
- Follows existing Data Cloud UI patterns and color schemes
- Red error modal header matches current error UI
- Warning modal uses orange/amber styling

### JavaScript Functionality
- `toggleGroup(header)` - Expands/collapses dependency group sections
- `switchToIntegrations()` - Programmatically switches to Integrations tab
- `closeModal(modalId)` - Closes modal dialogs
- `showErrorModal()` - Transitions from confirmation to error modal
- Event listeners for tab switching and delete button

### Modal States
- **Confirmation Modal**: Yellow/amber warning header, explains consequences of deletion
- **Error Modal**: Red error header, explains blocking dependencies, directs to Integrations tab

### Integrations Tab
- Dependency groups are collapsible with chevron indicators
- Each group shows count of dependencies
- Individual items display label (clickable) and API name
- Companion org references show badge with org name, non-clickable

## Making Changes

### Adding New Dependency Types
1. Add new dependency group in the Integrations tab section
2. Follow existing structure with header and items container
3. Use `dependency-item-name` class for clickable links
4. Use `dependency-item-name no-link` for non-clickable items

### Modifying Modal Behavior
- Modal overlay uses `show` class to display
- Header classes: `modal-header warning` (amber) or `modal-header error` (red)
- Footer contains action buttons (Cancel, Delete, View Integrations)

### Updating Dependency Data
Sample dependencies are hardcoded in HTML. In a real implementation, this would be populated from the API endpoint response.

## Design Reference

The mockup is based on the "Retriever Deletion: Show Dependencies with Navigable Links" design document which specifies:
- Separate sub-URL endpoint to avoid latency impact on main retriever GET
- Reuse of existing MlRetrieverDependencyHelper.fetchDependencies()
- Navigation URL construction for each dependency type
- Companion org reference enrichment from RemoteReferenceService
