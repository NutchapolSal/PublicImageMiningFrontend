web app for connecting to [PublicImageMining](https://github.com/NutchapolSal/aiproject#publicimagemining)

## Developing
install dependencies with `npm install` (or `pnpm install` or `yarn`)

then start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building
check your desired hosting path in `svelte.config.js` then run

```bash
npm run build
```

then copy the files in the `build` directory to any static webhoster