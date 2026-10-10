# @three-ws/see

<!-- three.ws:badges -->
[![GitHub stars](https://img.shields.io/github/stars/nirholas/three-ws-see?style=flat&logo=github)](https://github.com/nirholas/three-ws-see/stargazers) [![License](https://img.shields.io/github/license/nirholas/three-ws-see?style=flat)](https://github.com/nirholas/three-ws-see/blob/HEAD/LICENSE) [![Last commit](https://img.shields.io/github/last-commit/nirholas/three-ws-see?style=flat)](https://github.com/nirholas/three-ws-see/commits) [![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)](https://github.com/nirholas/three-ws-see/pulls) [![AI agent friendly](https://img.shields.io/badge/AI%20agents-AGENTS.md%20%2B%20llms.txt-6d5dfc?style=flat)](https://github.com/nirholas/three-ws-see/blob/HEAD/AGENTS.md)
<!-- /three.ws:badges -->


**Give an AI agent eyes for 3D.**

Every text-to-3D API on earth answers with a URL to a binary file. A human clicks it. An agent cannot: a `.glb` is opaque to a language model, so the agent that just generated an asset has no way to check its own work. That is why agentic 3D has been stuck at one shot and a shrug.

This package closes the loop in one call. Hand it a GLB URL and get back frames of the model rendered from several angles, ready to put in front of any multimodal model, plus the geometry facts and a plain reading of what they mean.

```
npm install @three-ws/see
```

No key. No account. The service is free.

## Look at a model

```js
import { see } from '@three-ws/see';

const look = await see('https://example.com/robot.glb');

look.views;
// [ { view: 'three-quarter', theta: 35, phi: 78, imageUrl: 'https://…/a.png' },
//   { view: 'front', … }, { view: 'side', … }, { view: 'back', … } ]

look.notes;
// [ '12,400 triangles, a normal real-time budget for a hero prop or character.' ]

look.stats;      // { vertices, triangles, materials, textures, animations, … }
look.viewerUrl;  // spin it in a browser
look.arUrl;      // place it in a real room from a phone
```

## Close the loop: generate, look, judge, fix

The reason this exists. An agent that can see its own output stops guessing.

```js
import Anthropic from '@anthropic-ai/sdk';
import { see, toMessageContent } from '@three-ws/see';

const anthropic = new Anthropic();

async function critique(glbUrl, intent) {
  const look = await see(glbUrl);
  const message = await anthropic.messages.create({
    model: 'claude-sonnet-5',
    max_tokens: 512,
    messages: [{
      role: 'user',
      content: [
        { type: 'text', text: `This model was meant to be: ${intent}. Judge it. Is the subject complete and recognisable, is the far side finished, is anything melted or fused? If it needs work, say exactly what to change.` },
        ...(await toMessageContent(look)),
      ],
    }],
  });
  return message.content[0].text;
}
```

`toMessageContent` shapes a look into multimodal chat content: one labelled text block per angle followed by the image itself. It passes image URLs by default; `{ fetchImages: true }` inlines base64 for APIs that will not fetch a URL themselves.

## Pick your angles

```js
await see(glbUrl, { views: ['back', 'top'], size: 768 });
```

| View | What it catches |
| --- | --- |
| `three-quarter` | Form and depth, the way a product shot reads (default first) |
| `front` | Facing, symmetry, the subject's identity |
| `side` | Profile, depth collapse, flattened geometry |
| `back` | The half a single-view generator most often leaves unfinished |
| `top` | Footprint, layout, whether it is hollow |
| `bottom` | Missing caps, inverted normals, floating geometry |

Up to six views per call, `size` from 128 to 1024 pixels (default 512). Unknown angle names are ignored rather than failing the call, so a guess still returns something useful.

## Errors are typed

```js
import { see, SeeError } from '@three-ws/see';

try {
  await see(url);
} catch (err) {
  if (err instanceof SeeError && err.code === 'rate_limited') {
    await new Promise((r) => setTimeout(r, err.retryAfter * 1000));
  }
}
```

`invalid_url`, `render_failed`, `rate_limited`, `timeout`, `unreachable`. A partially-rendered turntable is **not** an error: you get the frames that worked plus `look.missingViews` naming the angles that did not, because three good views beat a failure.

## Options

| Option | Default | Notes |
| --- | --- | --- |
| `views` | `['three-quarter','front','side','back']` | Up to 6, from the table above |
| `size` | `512` | Square pixels, 128 to 1024 |
| `stats` | `true` | Set false to skip the geometry pass |
| `baseUrl` | `https://three.ws` | Point at another deployment |
| `timeoutMs` | `120000` | Rendering holds a browser for seconds |
| `fetch` | global `fetch` | Inject for tests, proxies, tracing |

## Using MCP instead?

The same capability is an MCP tool, `look_at_model`, on `https://three.ws/api/mcp-studio`. There the frames come back as MCP **image content blocks**, so a multimodal client renders them straight into the conversation and the model sees the model with no glue code at all.

## Related

- [three.ws/docs/3d-vision](https://three.ws/docs/3d-vision): the full surface, REST and MCP
- [`POST /api/3d/look`](https://three.ws/api/3d/look): the endpoint underneath (GET it for a discovery doc)
- [`@three-ws/retarget`](https://www.npmjs.com/package/@three-ws/retarget): retarget animation onto any humanoid GLB

Apache-2.0.

<!-- three.ws:growth -->
## Support the project

If three-ws-see saves you time, **[star it on GitHub](https://github.com/nirholas/three-ws-see)**. Stars are how other developers and AI agents find the repositories worth trusting, and they cost you one click.

Know someone who would use it? [Post on X](https://twitter.com/intent/tweet?text=three-ws-see%3A%20Let%20an%20AI%20agent%20SEE%20a%203D%20model%3A%20render%20any%20GLB%20from%20several%20angles%20and%20get%20back%20frames%20it%20can&url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-see) · [Share on Bluesky](https://bsky.app/intent/compose?text=three-ws-see%3A%20Let%20an%20AI%20agent%20SEE%20a%203D%20model%3A%20render%20any%20GLB%20from%20several%20angles%20and%20get%20back%20frames%20it%20can%20https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-see) · [Share on LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-see) · [Submit to Hacker News](https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-see&t=three-ws-see%3A%20Let%20an%20AI%20agent%20SEE%20a%203D%20model%3A%20render%20any%20GLB%20from%20several%20angles%20and%20get%20back%20frames%20it%20can) · [Share on Reddit](https://www.reddit.com/submit?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fthree-ws-see&title=three-ws-see%3A%20Let%20an%20AI%20agent%20SEE%20a%203D%20model%3A%20render%20any%20GLB%20from%20several%20angles%20and%20get%20back%20frames%20it%20can)

## Built for AI agents too

Coding agents and LLM tooling can read this repo directly: [AGENTS.md](./AGENTS.md), [llms.txt](./llms.txt), [llms-full.txt](./llms-full.txt). Point an agent at `https://github.com/nirholas/three-ws-see` and it has the context it needs.

## More from the same author

- [All repositories by nirholas](https://github.com/nirholas/nirholas#readme): the full catalog, grouped by topic
- [three.ws](https://three.ws): the platform for 3D AI agents with Solana wallets, a skill marketplace and x402 payments
- Questions or ideas: [open an issue](https://github.com/nirholas/three-ws-see/issues) or [start a discussion](https://github.com/nirholas/three-ws-see/discussions)

## Contributors

[![Contributors](https://contrib.rocks/image?repo=nirholas/three-ws-see)](https://github.com/nirholas/three-ws-see/graphs/contributors)

## Star history

[![Star History Chart](https://api.star-history.com/svg?repos=nirholas/three-ws-see&type=Date)](https://www.star-history.com/#nirholas/three-ws-see&Date)

<!-- /three.ws:growth -->

<!-- three.ws:ecosystem -->
## Part of three.ws

[three.ws](https://three.ws) is a platform for 3D AI agents with Solana wallets: avatars, a skill marketplace, x402 payments and more than seventy MCP servers. `@three-ws/see` is one package from it.

- Documentation site: https://nirholas.github.io/three-ws-see/
- npm: https://www.npmjs.com/package/@three-ws/see
- Canonical source: https://github.com/nirholas/three.ws (this repository is a generated mirror of `packages/see`)
- Agent-readable summary: [llms.txt](./llms.txt) and [AGENTS.md](./AGENTS.md)
- Issues and ideas: https://github.com/nirholas/three-ws-see/issues
<!-- /three.ws:ecosystem -->
