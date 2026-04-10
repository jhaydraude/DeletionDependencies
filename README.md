# Retriever Deletion Dependencies Mockup

This interactive mockup demonstrates the improved user experience for deleting retrievers when dependencies exist.

## Features

### 1. Retriever Detail Page
- Overview tab showing retriever details (name, type, API name, data source, etc.)
- Integrations tab displaying all dependencies
- Delete Retriever button in the top-right corner

### 2. Delete Flow
1. **Confirmation Modal**: User clicks "Delete Retriever" and sees a warning confirmation
2. **Error Modal**: When dependencies exist, shows an improved error message directing users to the Integrations tab
3. **Integrations Tab**: Lists all dependencies organized by type with clickable navigation links

### 3. Dependency Types Shown
Based on the API response structure from the design document:

- **Agent Actions** - Clickable links to agent action setup pages
- **Flows** - Clickable links to flow builder
- **Prompt Templates** - Clickable links to template setup
- **Ensemble Retrievers** - Clickable links to Einstein Studio
- **Data Libraries** - Clickable links to library list page
- **Companion Org References** - Non-clickable, shows org name (e.g., "Sales Cloud LOB")

### 4. Interactive Elements
- Tab switching between Overview and Integrations
- Expandable/collapsible dependency groups
- Modal dialogs with proper close/cancel actions
- "View Integrations" button that automatically switches to the Integrations tab

## How to Use

1. Open `retriever-deletion-mockup.html` in a web browser
2. Click the "Delete Retriever" button to see the confirmation modal
3. Click "Delete" to see the error modal with the improved message
4. Click "View Integrations" or the link in the error message to navigate to the Integrations tab
5. Click on dependency group headers to expand/collapse the lists
6. Click on individual dependency names to simulate navigation (links are placeholder `#`)

## Design Notes

- Follows Salesforce Lightning Design System styling
- Uses the same visual patterns as existing Data Cloud UI
- Error modal uses red header (same as current implementation)
- Companion org references are visually distinguished with a badge and non-clickable name
- Dependency groups are collapsible for better organization
- All navigation URLs are placeholders (`#`) for demonstration purposes

## API Response Structure

The mockup is based on the `GET /ssot/ml/retrievers/{id}/integrations` endpoint structure documented in the design specification, which returns:

```json
{
  "references": [
    {
      "id": "...",
      "name": "...",
      "label": "...",
      "type": "Agent|Flow|PromptTemplate|EnsembleRetriever|DataLibrary|Other",
      "navigationUrl": "...",
      "isRemoteReference": true|false,
      "orgOwner": "Sales Cloud LOB" | null
    }
  ]
}
```
