# Composite Pattern (Recursive Tree)

**Workflow:** Recursive Tree Crawler
**RTC Relevance:** None

## Why This Pattern

Trees — file systems, nested comments, org charts, a GitHub repo's directory structure — have "leaf" and "container" nodes you often want to treat uniformly. Recursing into containers until you hit leaves is the Composite pattern's classic use case.

## Workflow Structure

1. Execute Workflow (self-reference) with input `{ path }`
2. Code node — fetch the directory listing for `path`
3. IF node — a directory entry triggers another Execute Workflow call (self) with that path; a file entry is collected directly
4. Code node — aggregate children's results with this level's own leaves

## Code Example

```javascript
const path = $json.path ?? '';
const repo = 'n8n-io/n8n';

const contents = await this.helpers.httpRequest({
  url: `https://api.github.com/repos/${repo}/contents/${path}`,
  headers: { 'User-Agent': 'n8n-practice' },
  json: true,
});

const files = contents.filter((entry) => entry.type === 'file');
const dirs = contents.filter((entry) => entry.type === 'dir');

// Each directory becomes a separate call to this same workflow (Execute Workflow, self-reference)
return dirs.map((dir) => ({ json: { path: dir.path, isDir: true } }))
  .concat(files.map((file) => ({ json: { path: file.path, isDir: false, size: file.size } })));
```

## Notes

The base case is simply "no subdirectories left" — `contents` for a leaf path returns no `dir` entries, so the recursion through Execute Workflow naturally terminates.
