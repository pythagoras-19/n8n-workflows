# Visitor Pattern (Graph Traversal)

**Workflow:** Graph Traversal Workflow
**RTC Relevance:** None

## Why This Pattern

BFS and DFS both "visit" every node in a graph, but what you *do* at each node — check for a target, mark visited, accumulate a path — varies by use case. Separating traversal order from the visit action is exactly the Visitor pattern's idea.

## Workflow Structure

1. Code node — define the graph as JSON
2. Code node — generic traversal function, parameterized by a "visit" callback

## Code Example

```javascript
function bfs(graph, start, visit) {
  const queue = [[start, [start]]];
  const seen = new Set([start]);

  while (queue.length) {
    const [node, path] = queue.shift();
    const result = visit(node, path);
    if (result?.stop) return result;

    for (const neighbor of graph[node] ?? []) {
      if (!seen.has(neighbor)) {
        seen.add(neighbor);
        queue.push([neighbor, [...path, neighbor]]);
      }
    }
  }
  return null;
}

function detectCycle(graph) {
  const visiting = new Set();
  const visited = new Set();

  function dfs(node) {
    visiting.add(node);
    for (const neighbor of graph[node] ?? []) {
      if (visiting.has(neighbor)) return true; // back edge = cycle
      if (!visited.has(neighbor) && dfs(neighbor)) return true;
    }
    visiting.delete(node);
    visited.add(node);
    return false;
  }

  return Object.keys(graph).some((node) => !visited.has(node) && dfs(node));
}

const graph = $json.graph ?? {
  A: ['B', 'C'], B: ['D'], C: ['D'], D: ['E'], E: [],
};

const shortestPathToE = bfs(graph, 'A', (node, path) =>
  node === 'E' ? { stop: true, path } : null
);

return [{ json: { shortestPathToE: shortestPathToE?.path, hasCycle: detectCycle(graph) } }];
```

## Notes

`bfs`'s `visit` callback is the "visitor" — swap it for a different callback (e.g. one that just collects every node) and the traversal logic itself doesn't change at all.
