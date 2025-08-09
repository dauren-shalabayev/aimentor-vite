# aimentor

This template should help get you started developing with Vue 3 in Vite.

## Environment Variables

This project requires the following environment variables to be set in a `.env` file:

```bash
# ChatGPT API Key for answer validation
VITE_OPENAI_API_KEY=your_openai_api_key_here
```

### Setting up environment variables:

1. Create a `.env` file in the root directory
2. Add your OpenAI API key:
   ```
   VITE_OPENAI_API_KEY=sk-proj-your-api-key-here
   ```
3. The `.env` file is already in `.gitignore` to keep your API key secure

**Security Note:** Never commit your actual API keys to version control. The `.env` file is automatically ignored by Git to prevent accidental exposure of sensitive information.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
