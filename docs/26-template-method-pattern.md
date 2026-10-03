# Template Method Pattern

**Workflow:** Custom Template Engine
**RTC Relevance:** None

## Why This Pattern

Generating HTML from data by string concatenation gets unreadable fast. A small template renderer with variable substitution, loops, and conditionals gives you a reusable "algorithm skeleton" — parse, then render — that different templates plug into.

## Workflow Structure

1. Code node — template renderer + report data
2. Email/Telegram node — send the rendered output

## Code Example

```javascript
function render(template, data) {
  // {{#each items}}...{{/each}}
  template = template.replace(/{{#each (\w+)}}([\s\S]*?){{\/each}}/g, (_, key, inner) => {
    return (data[key] ?? []).map((item) => render(inner, { ...data, ...item })).join('');
  });

  // {{#if cond}}...{{/if}}
  template = template.replace(/{{#if (\w+)}}([\s\S]*?){{\/if}}/g, (_, key, inner) => {
    return data[key] ? render(inner, data) : '';
  });

  // {{variable}}
  template = template.replace(/{{(\w+)}}/g, (_, key) => data[key] ?? '');

  return template;
}

const template = `
<h1>Daily Digest for {{date}}</h1>
{{#if hasStories}}
<ul>
{{#each stories}}
  <li>{{title}} ({{points}} points)</li>
{{/each}}
</ul>
{{/if}}
`;

const data = {
  date: new Date().toDateString(),
  hasStories: ($json.stories ?? []).length > 0,
  stories: $json.stories ?? [],
};

return [{ json: { html: render(template, data) } }];
```

## Notes

The "template method" here is `render` itself — it always does substitution → conditionals → loops in the same order regardless of which template string you hand it, which is the fixed algorithm skeleton the pattern is named for.
