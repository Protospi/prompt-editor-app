# Prompt Editor App

A sophisticated Markdown-based prompt editor for interacting with AI models. This application allows users to create, format, and manage prompts with dedicated formatting options for AI interactions.

## Features

- **Markdown Output**: Creates clean Markdown output that can be sent directly to AI models
- **Rich Text Editing**: Format text with bold, italic, and lists
- **Title Formatting**: Format any line as a title (H6) with a single click
- **Toggle Format**: Click on a formatted title to revert it to normal text
- **Line-by-Line Control**: Each line maintains its own independent formatting
- **Live Preview**: See how your formatted prompt will appear
- **Tab Indentation**: Support for tab indentation in lists

## Usage

The editor features a simple toolbar with the following options:

- **Bold**: Format selected text as bold
- **Italic**: Format selected text as italic
- **Bullet List**: Create an unordered list
- **Title (Tt)**: Format the current line as a title
- **Paragraph (P)**: Format the current line as normal paragraph text

## Technical Details

This application is built with:

- **Vue 3**: For the reactive UI
- **Quasar Framework**: For UI components and styling
- **TypeScript**: For type safety and better developer experience
- **Turndown**: For converting HTML to Markdown

The editor uses a contenteditable div with proper Markdown conversion to ensure reliable output formatting.

## Installation

```bash
# Install dependencies
yarn install
# or
npm install

# Start the development server
quasar dev

# Build for production
quasar build
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

[MIT](LICENSE)
