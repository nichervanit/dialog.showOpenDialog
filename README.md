What changed — full replacement for shell/browser/ui/file_dialog_mac.mm:

	•	SetAllowedFileTypes → SetAllowedContentTypes, using NSSavePanel.allowedContentTypes (UTType[]) instead of the deprecated setAllowedFileTypes: (strings).
	•	For each extension, it now calls +[UTType typesWithTag:tagClass:conformingToType:] and adds every matching UTType — not just the first — so app-declared package types (TextEdit’s .rtfd bundle type, etc.) are included alongside the generic public type.
	•	PopUpButtonHandler (the multi-filter format picker) updated to the same API.
	•	"*" wildcard now maps to setAllowedContentTypes:@[] (Apple’s documented “allow everything” value for this API, cleaner than the old nil-trick).
	•	Kept a @available(macOS 11.0, *) fallback to the old string-based path for pre-Big Sur, since I couldn’t confirm Electron 36’s exact minimum deployment target from here.

Before you build:

	1.	Add UniformTypeIdentifiers.framework to the frameworks list for the target that compiles this file (check shell/browser/BUILD.gn — Electron already links Cocoa/CoreServices there, this needs to sit next to them). If the linker complains about undefined OBJC_CLASS_$_UTType, that’s the fix.
	2.	Drop this file in over the original and rebuild just the electron target — no gclient resync needed since it’s a pure source change.

Please verify on your Mac (I have no macOS/Xcode here to compile this):

	•	Repro from the gist with the .rtfd package — should now be selectable.
	•	Regression-check the existing fix for #46883/#46900 (single-filter case still applies filters).
	•	Regression-check .app filtering still works.
	•	Multi-filter format picker still switches correctly.
	•	Save dialogs (not just open) with filters.
