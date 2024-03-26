# Markdown Oxide

Implementing Obsidian PKM features (and possibly more) in the form of a language server allows us to use these features in our favorite text editors and reuse other LSP-related plugins (like Telescope, outline, refactoring tools, ...). This language server primarily attempts to be a replacement and extension to the Obsidian markdown editor for Obsidian vault PKM-ing in your favorite text editing system -- creating the best PKM system for any text editor. 

## Installation

> [!IMPORTANT]
> I'm working on getting this into package distributions. Installation/configuration should be easier soon. Also this is very pre-release and things may be broken


### Arch Linux

If you are using Arch Linux, you can install the latest Git version through the AUR. The package name is `markdown-oxide-git`. You can install it using your preferred AUR helper; for example:

```sh
paru -S markdown-oxide-git
```

### Cargo (Linux, MacOS, Windows)

If you have cargo installed, you can easily install the binary for the LS by running the following command:

```sh
cargo install --git https://github.com/Feel-ix-343/markdown-oxide.git markdown-oxide
```

### Manual (MacOS, Windows, Linux)

Clone the repository and then run `cargo build --release`.

You will subsequently need the path to the release binary when you configure your editor. It can be found relative to the root of the project at `target/release/markdown-oxide`

## Usage

To use the language server, you need to follow the instructions for your editor of choice below.

### VSCode

Go to [the VSCode extension readme](./vscode-extension/README.md) and run the commands listed

### Neovim

Make sure rust is installed properly and that you are using nvim cmp (I am not sure if it works in other completion engines)

Adjust your neovim config as follows

```lua
local lspconfig = require('lspconfig')
local configs = require("lspconfig.configs")

require("lspconfig").markdown_oxide.setup({
    capabilities = capabilities -- ensure that capabilities.workspace.didChangeWatchedFiles.dynamicRegistration = true
    root_dir = lspconfig.util.root_pattern('.git', vim.fn.getcwd()), -- this is a temp fix for an error in the lspconfig for this LS
})
```

then adjust your nvim-cmp source settings for the following. Note that this will likely change in the future.

```lua
{
name = 'nvim_lsp',
  option = {
    markdown_oxide = {
      keyword_pattern = [[\(\k\| \|\/\|#\)\+]]
    }
  }
},
```


I also recommend enabling codelens in neovim. Add this snippet to your on\_attach function for nvim-lspconfig


```lua
-- refresh codelens on TextChanged and InsertLeave as well
vim.api.nvim_create_autocmd({ 'TextChanged', 'InsertLeave', 'CursorHold', 'LspAttach' }, {
    buffer = bufnr,
    callback = vim.lsp.codelens.refresh,
})

-- trigger codelens refresh
vim.api.nvim_exec_autocmds('User', { pattern = 'LspAttached' })
```


*Test it out! Go to definitions, get references, and more!*

> [!NOTE]
> To get references on files, you can have your cursor anywhere on the markdown file where there is not another referenceable (heading, tag, ...)

## Note on Linking Syntax

The linking syntax is that of Obsidian's and can be found here https://help.obsidian.md/Linking+notes+and+files/Internal+links

Generally, this is `[[relativeFilePath(#heading)?(|display text)?]]` e.g. [[articles/markdown oxide#Features|Markdown Oxide Features]] to link to a heading in `Markdown Oxide.md` file in the `articles` folder or [[Obsidian]] for the `Obsidian.md` file in the root folder. Markdown oxide also support markdown links

## Features

### Completions

- <details>
  <summary>Wikilink Completions</summary>

  ![wikilinkcompletions](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/29c4830f-30e5-4094-9f5b-7b39009437da)
  
</details>

- <details>
    <summary>Markdown Link Completions</summary>

    ![markdownlinkcompletions](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/16c8565a-6a28-4df1-a312-e4b158fb9f03)

    
</details>

- <details>
    <summary>Unindexed Block Completions: Make the linking syntax and press space; Fuzzy searches through the whole folder of files and allows you to link anywhere, following obsidian block linking syntax</summary>

    ![blockcompletions](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/a48c28a7-55b0-438c-becc-1dfde350fa94)
    
</details>


- <details>
    <summary>Tag Completions</summary>

    ![tagcompletions](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/bf20d7ac-171a-4d95-b510-ba323073c0b8)

    
</details>

- <details>
    <summary>Footnote Completions</summary>

    ![footnotecompletions](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/92a6739d-8a7a-457e-84bd-fde6548aa25a)
    
</details>

### References

- <details>
    <summary>File References: Gets references to the file and all headings and blocks in the file</summary>

    ![filereferences](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/9fbd6051-ef57-42eb-b61b-1cc3ddfb2293)
    
</details>

- <details>
    <summary>Heading References</summary>

    
    ![headingreferences](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/50598628-ed27-4a9b-adba-861ca8f933ea)
    
</details>

- <details>
    <summary>Tag References: Gets all references to the tag and subtags</summary>

    ![tagreferences](https://github.com/Feel-ix-343/markdown-oxide/assets/88951499/d73ac764-2c86-45c9-9403-17b50e6962e4)
    
</details>



- Get references
    - [X] For File when the cursor is anywhere where there is not another referenceable. This will produce references not only to the file but also to headings and blocks in the file
    - [X] For block when the cursor is on the block's index "...text *^index*"
    - [X] For tag when the cursor is on the tags declaration. Unlike go-to-definition for tags, this will produce all references to the tag and to the tag with subtags
    - [X] Footnotes when the cursor is on the declaration line of the footnote; *[^1]: description...*

> [!NOTE]
> I strongly recommend using [Lspsaga](https://github.com/nvimdev/lspsaga.nvim) for references for two reasons. First because this LS sorts references by the date their files were modified and unlike `vim.lsp.buf.references()` and `Telescope lsp_references`, `Lspsaga finder` maintains this sorting order. Second it also allows you to edit the references in place, similar to Logseq

- Completions (requires extra nvim-cmp config; follow the directions above)
    - [X] File link completions
    - [X] Heading link Completions
    - [ ] Subheading completions in the form [[file#heading#subheading]] from https://help.obsidian.md/Linking+notes+and+files/Internal+links#Link+to+a+heading+in+a+note (Note: right now you can link to subheadings through [[file#subheading]])
    - [X] Block link completions (searches the text of the indexed block) 
    - [X] Footnote link completions
    - [X] New Block link Completions through grep: to use this, type `[[`, and after you press space, completions for every block in the vault will appear; continue typing to fuzzy match the block that you want; finally, select the block; a link will be inserted to the text document and an index (ex ^1j239) will be appended to the block in its respective file. In Neovim, this text will not be written yet into the file (it will be edited in an unsaved buffer) so type `:wa`, and it should be resolved (as long as you have `dynamicRegistration = true` as described [here](https://github.com/Feel-ix-343/markdown-oxide?tab=readme-ov-file#neovim)!
    - [ ] Callout/admonition completions
    - [ ] Metadata completions
    - [ ] Dataview completions
    - [ ] Metadata tag completions
    - [ ] \`\`\`query\`\`\` code block completions
- Hover Preview
    - [X] File
    - [X] Headings
    - [X] Indexed Blocks
    - [X] Footnotes
    - [X] Show backlinks, sorted by the date modified, in the hover (I will write most of the content for a note not in the note itself, but in backlinks to the note; I also will write in notes at times. This feature is to combine the content related to the note including both backlinks in actual organized text)
- [ ] Code Actions
    - [x] Unresolved file link -> Create the file
    - [x] Unresolved heading link -> append heading to file and create the file if necessary
    - [ ] Link suggestions (by text match or other)
    - [ ] Refactoring: Move headers or selections to a new file
    - [ ] Link an unlinked reference
    - [ ] Link all unlinked references to a referenceable
- [X] Diagnostics
    - [X] Unresolved reference
    - [ ] Unlinked reference
- [X] Symbols
    - [X] File symbols: Headings and subheadings
    - [X] Workspace headings: everything linkable: files, headings, tags, ... Like a good search feature
    - [ ] Lists and indented lists
- [ ] Rename
    - [X] File (cursor must be in the first character of the first line)
    - [X] Headings
    - [X] Tags
    - [ ] Indexed Blocks
- [ ] Dataview support
- Config
    * [ ] Daily notes format
- A simple CLI
    - [ ] Working with daily notes (key to efficient PKM systems!)
    - [ ] ... (leave some ideas in the issues!)
- [ ] Integrate with Obsidian.nvim


# Config

`Markdown-Oxide` supports several configuration options. All can be specified in a `~/.config/moxide/settings.toml` or `.moxide.toml` file and moxide tries to import some settings (daily notes formatting) from Obsidian directly. Here are the options with the defaults

```toml
# Leave blank to try to import from Obsidian Daily Notes
# Formatting from https://docs.rs/chrono/latest/chrono/format/strftime/index.html
dailynote = "%Y-%m-%d" # this is akin to YYYY-MM-DD from Obsidian

# Fuzzy match file headings in completions
heading_completions = true

# Set true if you title your notes by the first heading
# Right now, if true this will cause completing a file link in the markdown style
# to insert the name of the first heading in the display text area
# [](file) -> [first heading of file.md](file)
# If false, [](file) -> [](file) (for example)
title_headings = true

# Show diagnostics for unresolved links; note that even if this is turned off, 
# special semantic tokens will be sent for the unresolved links, allowing you
# to visually identify unresolved links
unresolved_diagnostics = true
```


# Alternatives

**I love open-source and all open-source authors!! I also believe healthy competition is good! Markdown-Oxide is competing with some alternatives, and I want to make it the best at its job!!**

Here are the alternatives (open source authors are welcome to make PRs to add their projects here!)

- https://github.com/gw31415/obsidian-lsp: I have been in discussions with the author; The author doesn't have time to maintain the project. Also, I, of course, love the idea, but the current LS doesn't provide many obsidian-specific features yet.
- https://github.com/WhiskeyJack96/logseqlsp: This is a cool project and a great inspiration for Logseq support (which is upcoming). status: it doesn't seem that it is maintained and it (obviously) does not provide support for all of the obsidian syntax
- The og https://github.com/artempyanykh/marksman: I used this for a while, but it is not obsidian specific and didn't act well with my vault. Additionally, the block completions in markdown-oxide allow for a fuzzy/grep search of the entire vault to generate the completions; I don't think Markman has any features like this; (this is a feature that Logseq signified for PKM; the concept that *anything is linkable* is quite powerful) 


# ---The--bottom--line--------------------------------------------------

Listen. I really like Vim motions. I also really love low-latency terminal editing. I very much so also like my Neovim LSP plugins, keymappings, and config. But Wow! I also like using Obsidian and Logseq. **Can't I just have it all???** Can't I be whisked away by the flow of Neovim while also experiencing the beauty of Obsidian???? Can't I detail my tasks on the CLI while viewing them in Logseq????? Well, I thought I could; for us all, there is markdown-oxide (which is still very pre-beta hah)
