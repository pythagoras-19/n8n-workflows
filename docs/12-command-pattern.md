# Command Pattern (Real-Time Chat Bot)

**Workflow:** Telegram Command Bot
**RTC Relevance:** High — live two-way chat

## Why This Pattern

An `if/else if` chain of commands gets unwieldy fast. The Command pattern turns each command into an entry in a lookup table (a function), so adding a command means adding one registry entry, not touching existing branches.

## Workflow Structure

1. Telegram Trigger — on message
2. Code node — parse `/command args` and dispatch via a registry
3. Telegram node — send the reply

## Code Example

```javascript
const commands = {
  async weather(args) {
    const city = args.join(' ') || 'New York';
    return `Weather lookup for ${city} is not wired up yet — swap in a real API call here.`;
  },
  async quote() {
    const res = await this.helpers.httpRequest({ url: 'https://zenquotes.io/api/random', json: true });
    const [{ q, a }] = res;
    return `"${q}" — ${a}`;
  },
  async help(_, registry) {
    return `Available commands: ${Object.keys(registry).join(', ')}`;
  },
};

const text = $json.message?.text ?? '';
const [rawCommand, ...args] = text.trim().split(/\s+/);
const commandName = rawCommand.replace('/', '').toLowerCase();

const handler = commands[commandName] ?? commands.help;
const reply = await handler.call(this, args, commands);

return [{ json: { chatId: $json.message.chat.id, reply } }];
```

## Notes

Each registry entry is a self-contained "command object" — the dispatcher never needs to know what a command does, only that it's callable, which is the essence of the pattern.
