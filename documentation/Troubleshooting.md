## Troubleshooting guide

This is a collection of cases which have been reported and discussed on the https://github.com/eclipse/corrosion/issues GitHub tracker.

### Components and where to find them

The Eclipse IDE for Rust Developers builds mainly on four subprojects:

- The basic Eclipse IDE platform
  - https://www.eclipse.org/downloads/packages/

- The Corrosion Rust development plugin
https://www.eclipse.org/downloads/packages/release/2018-09/r/eclipse-ide-rust-developers-includes-incubating-components

https://github.com/eclipse/corrosion - this repository
https://github.com/eclipse/corrosion/issues 
https://github.com/eclipse/corrosion.git

Corrosion Snapshots - http://download.eclipse.org/corrosion/snapshots/
Corrosion Releases - http://download.eclipse.org/corrosion/releases/

- The TextMate plugin which provides basic language-aware text editing 

https://github.com/eclipse/tm4e
https://github.com/eclipse/tm4e/issues
https://github.com/eclipse/tm4e.git

https://projects.eclipse.org/projects/technology.tm4e
TM4E Snapshots - http://download.eclipse.org/tm4e/snapshots/

- Language Server Protocol for Eclipse (LSP4E)

LSP4E Snapshots - http://download.eclipse.org/lsp4e/snapshots/

### Gitter chat

https://gitter.im/eclipse_rust_development/Corrosion

### rls.conf

Hi @theDragonFire: the `.cargo/rls.conf` file contains the startup settings for the `rls` and it is optional. If you don't have one, Corrosion will ignore it and guess some sensible (!) defaults. 

In this file you should be able to specify any of the valid RLS settings, as listed in https://github.com/rust-lang/rls#configuration. Syntax for these options is IDE-specific: the PR at https://github.com/eclipse/corrosion/pull/183 contains an example config file for Corrosion.

Unfortunately, none of the settings in RLS, as I can understand, allow you to specify `rustc` path, which should be set up for you by `rustup` (check your env vars?) or `racer` - which AFAIK is linked into the RLS as a library rather than as an external tool.



### Enable RLS logging

Language Server log to file can be enabled for the the Rust Language server here:

![image](https://user-images.githubusercontent.com/889291/49970871-9b0c9a00-ff24-11e8-92c8-16886f4b7b08.png)

Once the logging is active, and you have restarted your IDE, Eclipse will output some RLS logging here:

`path/to/your/eclipse_rust_workspace/languageServers-log/org.eclipse.corrosion.rls.log`

### Even more logging

Error messages that are enabled via the environment variable `RUST_LOG` are written to std-err.

In LSP4E std-err outputs are written to the language server log. This mixes both regular LSP JSON messages with std-err outputs. For me personally, this is not a big issue, if you care for such separation, please open an issue on the LSP4E project.

As the `RUST_LOG` can't be (yet) set from inside the Rust configuration dialog in the IDE, it has to be set externally, before starting Eclipse IDE.

#### Examples:

Linux:
- open a terminal window
- type:
```
export RUST_LOG=rls=debug
/path/to/my/eclipse&
```

Windows:
- add `RUST_LOG` environment variable from the Environment Variables system dialog, with value `rls=debug`
- restart Eclipse


### Other source of logs

Some useful diagnostic messages are sent to the Eclipse `Error log` view:

![image](https://user-images.githubusercontent.com/889291/49970820-6c8ebf00-ff24-11e8-8bc2-72ab4c92ece4.png)

## Common issues

### Multiple duplicate highlight icons in perspective toolbar

Occasionally, the toolbar spawns an extra highlight icon. After some time you can end up with a crowded toolbar, like this:  

![crowded_toolbar](https://user-images.githubusercontent.com/889291/48922009-88f68900-ee9b-11e8-9b10-93764e254ab1.png)

This is a long known issue with the eclipse platform, reported a https://bugs.eclipse.org/bugs/show_bug.cgi?id=534325

GitHub Corrosion Reference: https://github.com/eclipse/corrosion/issues/120

#### Workaround

1. Close the Eclipse IDE
2. open `path/to/workspace_rust/.metadata/.plugins/org.eclipse.e4.workbench/workbench.xmi` in a text editor
3. Locate the section below and locate the duplicate `menu:HandledToolItem`
```xml
<children xsi:type="menu:ToolBar" xmi:id="_CsPCkrN_EeicRcvSpkqvKw" elementId="org.eclipse.ui.edit.text.actionSet.presentation">
        <tags>Draggable</tags>
        <children xsi:type="menu:HandledToolItem" xmi:id="_CsPCk7N_EeicRcvSpkqvKw" elementId="org.eclipse.ui.genericeditor.togglehighlight" visible="false" iconURI="platform:/plugin/org.eclipse.ui.genericeditor/icons/full/etool16/mark_occurrences.png" type="Check" command="_CsXlFrN_EeicRcvSpkqvKw">
          <persistedState key="IIdentifier" value="org.eclipse.ui.genericeditor/org.eclipse.ui.genericeditor.togglehighlight"/>
          <visibleWhen xsi:type="ui:CoreExpression" xmi:id="_CsPClLN_EeicRcvSpkqvKw" coreExpressionId="programmatic.value"/>
        </children>
        <children xsi:type="menu:HandledToolItem" xmi:id="_CsPClbN_EeicRcvSpkqvKw" elementId="org.eclipse.ui.genericeditor.togglehighlight" iconURI="platform:/plugin/org.eclipse.ui.genericeditor/icons/full/etool16/mark_occurrences.png" type="Check" command="_CsXlFrN_EeicRcvSpkqvKw">
          <persistedState key="IIdentifier" value="org.eclipse.ui.genericeditor/org.eclipse.ui.genericeditor.togglehighlight"/>
          <visibleWhen xsi:type="ui:CoreExpression" xmi:id="_CsPClrN_EeicRcvSpkqvKw" coreExpressionId="programmatic.value"/>
        </children>
[...snip...]
```
4. Manually delete `<children xsi:type="menu:HandledToolItem" .../?` elements in excess, leaving only one copy
5. Save and exit
6. Start Eclipse Ide

### Cannot place breakpoint in library code
### Cannot browse/Ctrl+click/go-to-definition in library code 

In order for a Rust breakpoint to be placed in a source file, the source file must be part of the workspace, and that includes all the files in the `.cargo` cache.

Corrosion should automatically import the `.cargo` cache in the workspace as a project if it doesn't find it.

Should this process fail, some useful functionality will not work.

#### Workaround

Manually import the `.cargo` folder as a project in your Rust IDE workspace by:

1. Select **File** -> **Open projects from File System...** from the main menu
2. Click on **Directory...*** and select the `.cargo` folder from your home directory (or wherever you have placed your `.cargo`)
3. **Import source** `/your/home/path/.cargo` should be selected
4. Click on **Finish**

### Missing field 'codeActionKind'.

> org.eclipse.lsp4j.jsonrpc.ResponseErrorException: missing field 'codeActionKind'.

This was caused by a bug in the Language Server Protocol handler in Eclipse, fixed in:

https://bugs.eclipse.org/bugs/show_bug.cgi?id=541851
https://git.eclipse.org/c/lsp4e/lsp4e.git/commit/?id=6cc786f0d9bc1c12b818f7677796e2b4efdefbef
https://github.com/rust-lang/rls/issues/1161

#### Fix

Installing/upgrading to an up-to-date version of Eclipse IDE for Rust Developers such as 2018-09 or 2018-12, or manually updating from the Snapshot update sites, will resolve it.

## Living on the bleeding edge

- make sure you have a recent/working toolchain with RLS (tested with nightly 2018-12-13)
- download and install Eclipse IDE For Rust Developer from https://www.eclipse.org/downloads/packages/release/2018-09/r/eclipse-ide-rust-developers-includes-incubating-components
- start the installed Eclipse. This will not work out of the box and will exhibit the problem you reported with `codeActionKind`. Ignore it for now.
- Help -> Install new software

Do not press `Next >`. Instead, add the following three sites via `Add...` or `Manage...`

Corrosion Snapshots - http://download.eclipse.org/corrosion/snapshots/
TM4E Snapshots - http://download.eclipse.org/tm4e/snapshots/
LSP4E Snapshots - http://download.eclipse.org/lsp4e/snapshots/

- Close the "Install" window
- Help -> Check for updates
- accept the proposed selection (in my case Corrosion wanted to upgrade)
- press Next/Finish to the end
- restart Eclipse when prompted
