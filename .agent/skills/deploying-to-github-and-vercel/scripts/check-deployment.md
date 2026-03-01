# Vercel MCP Deployment Status Queries

Use these Vercel MCP tool calls to check deployment status after a GitHub push.

## 1. List all projects (to find projectId)
```
mcp_vercel-mcp_vercel_list_projects()
```
Look for your project by `name` field. Copy its `id` (starts with `prj_`).

## 2. List latest deployments for a project
```
mcp_vercel-mcp_vercel_list_deployments(
  projectId: "prj_xxxxxxxxxxxxxxxxxxxx",
  limit: 3
)
```

### Response fields to check:
| Field | Meaning |
|---|---|
| `readyState` | `BUILDING`, `READY`, `ERROR` |
| `readySubstate` | `PROMOTED` = live in production |
| `alias` | Array of live URLs |
| `inspectorUrl` | Link to Vercel build logs |
| `meta.githubCommitMessage` | Confirms which commit triggered this |

## 3. Force redeploy (if auto-deploy didn't trigger)
```
mcp_vercel-mcp_vercel_redeploy(
  deploymentId: "dpl_xxxxxxxxxxxxxxxxxxxx",
  target: "production"
)
```
Get `deploymentId` from the `uid` field in `list_deployments` response.
