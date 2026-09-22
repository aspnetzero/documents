# Debugging ABP Packages (Symbol Server)

Starting with **.NET 10**, [ASP.NET Boilerplate](https://aspnetboilerplate.com/) (ABP) NuGet packages (`Abp`, `Abp.AspNetCore`, `Abp.ZeroCore`, `Abp.Zero.Common`, etc.) are not published on [nuget.org](https://www.nuget.org/) anymore. They are distributed on ASP.NET Zero's private NuGet feed and are available to ASP.NET Zero license owners:

```
https://nuget.aspnetzero.com/{feedKey}/v3/index.json
```

The packages themselves don't contain PDB files, but **symbol packages are published for them** on the same feed. Symbols are served from a symbol server address that is different from the package feed address, so you have to add that address to your debugger once. After that, you can set breakpoints in the ABP code, see real local variables and get complete call stacks while debugging your own application.

## Symbol Server Address

```
https://nuget.aspnetzero.com/{feedKey}/api/download/symbols
```

`{feedKey}` is the same key that appears in your package feed address, so you can simply replace `v3/index.json` with `api/download/symbols` in the address you already use to restore packages. You can find your feed address on [aspnetzero.com/LicenseManagement](https://aspnetzero.com/LicenseManagement).

## Visual Studio

1. Open **Tools → Options → Debugging → Symbols**.
2. Click the **+** (New Location) button and enter the symbol server address above.
3. Set a **Cache symbols in this directory** folder if you don't have one yet, so that symbols are downloaded only once.
4. Open **Tools → Options → Debugging → General** and **uncheck** *Enable Just My Code*. Otherwise the debugger doesn't step into code outside of your own solution.
5. Click **OK** and start debugging.

The first request to the symbol server can take a while. If you don't want to wait for the symbols of every module, leave the symbol location unchecked in the options page and load symbols on demand instead: while debugging, open **Debug → Windows → Modules**, right-click the `Abp.*` module you are interested in and select **Load Symbols**.

> **Note:** Only symbols are published, not the source code. With symbols loaded you get real method names, parameters, local variables and line-accurate call stacks. When you step into a method, Visual Studio shows the decompiled source of the assembly.

## Other Debuggers

The steps above are for Visual Studio. The address is a standard NuGet symbol server address, so any other debugger that supports custom symbol servers can use the same address in its own symbol settings.

## Troubleshooting

* **How do I know if the symbols are loaded?** While debugging, open **Debug → Windows → Modules** and check the **Symbol Status** column of the `Abp.*` modules. It should be *Symbols loaded*.
* **Symbols are downloaded but the debugger doesn't step into the code.** Check that *Enable Just My Code* is disabled and that your solution is built in the **Debug** configuration. If the symbols are loaded and stepping still doesn't work, the cause is usually local debugger configuration rather than the symbol server.
* **Symbols aren't downloaded at all.** Verify that the `{feedKey}` in the symbol server address is the same key as in your package feed address, and that your license is still active. The symbol server requires a valid feed key exactly like the package feed does.
* **An old symbol file is used.** Clear your symbol cache directory (the folder configured in **Tools → Options → Debugging → Symbols**) and start a new debugging session.
