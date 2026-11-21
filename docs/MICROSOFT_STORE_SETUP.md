# Microsoft Store route for GIZE AI Windows
Decision: 21 November 2026. Owner selected Microsoft Store MSIX distribution; no developer account or reserved product exists yet.

Status: preparation only. No MSIX has been built, signed, submitted, certified or installed. Existing unsigned developer-preview builds remain the current technical artifacts. Production release gates remain unchanged.

## Owner account setup
1. Start at https://storedeveloper.microsoft.com/ and use the new onboarding flow.
2. For this commercial Orgaiz/GIZE AI product, choose Company and provide the correct legal publisher. Microsoft lists business/professional distribution under Company; do not choose a hobbyist account just to avoid business verification.
3. Prepare business verification documents or a D-U-N-S number and a work email associated with the business domain. Complete identity/business verification and personally review the developer agreement. Do not place documents, credentials, ID images or signing secrets in GitHub or this checklist.
4. After verification, open Partner Center > Apps & Games > New product > MSIX or PWA app.
5. Check availability and reserve the desired GIZE AI product name. The display name is a request, not a confirmed reservation.
6. Open Product identity and supply only the non-secret Package/Identity/Name, Package/Identity/Publisher and Package/Properties/PublisherDisplayName values to the packaging engineer. Do not guess these from a GitHub username or domain.

Microsoft's new onboarding documentation currently states that both account types have no registration fee when entered from the Store developer site. If a fee appears, check that this is the documented new flow before committing payment. Free MSIX signing is part of the Store distribution path; it does not sign the existing standalone EXE.

## Packaging engineering before submission
The existing application is a Chrome extension and .NET native bridge, not a Store-ready desktop package:
- `tools/TheOrg-Start.ps1` expects a dedicated bundled Chrome for Testing browser, loads the unpacked extension, discovers its ID, and invokes `install-native-host.ps1`.
- `tools/install-native-host.ps1` writes the host JSON under LocalAppData and registers HKCU\\Software\\Google\\Chrome\\NativeMessagingHosts\\com.theorg.bridge. A packaged process's successful registry write does not prove that unpackaged Chrome can see it.
- `engine/bridge/TheOrg.Bridge.csproj` builds the stdio host. It is not a replacement for an interactive desktop entry point.
- `tools/TheOrgPreview.nsi` owns the current EXE install/uninstall lifecycle. Wrapping it or renaming its output to .msix is insufficient.

Required work:
1. Build a desktop MSIX entry point with an identity taken from Partner Center, appropriate assets, versioning and only justified capabilities.
2. Test Chrome host discovery from the chosen packaged/dedicated browser and from Chrome Stable separately. Evaluate narrow Windows 11 virtualization exclusions only where needed; restricted capabilities require justification and review. Do not disable virtualization globally or make policy exceptions merely to obtain a passing check.
3. Verify read-only package paths, writable profiles, SQLite/workspaces, encrypted-artifact paths and host executable launch under package identity.
4. Design update and uninstall lifecycle handling for externally visible native-host registration, including stale paths after package-version changes. Retain ownership checks; never delete a newer install's registration. MSIX uninstall does not run the NSIS uninstaller.
5. Verify browser licensing, update strategy and extension distribution. The bundled pinned testing browser is not a validated production auto-update solution.
6. Test migration/coexistence with the old preview without deleting company data or reusing the everyday browser profile.

## Acceptance and submission
- Install the actual MSIX candidate on a clean test Windows device; test launch, Chrome connection, restart, upgrade, uninstall and data retention.
- Run Windows App Certification Kit and record results against the exact package hash/source commit. Synthetic native-stdio tests alone do not establish physical Chrome acceptance.
- Prepare accurate listing text, current screenshots, support details, privacy policy URL, age ratings and restricted-capability explanations.
- The owner reviews the final listing and submission. Microsoft certification/Store delivery provides the Store-distributed package signature.
- After certification, install from the Store and verify the delivered identity, version, signature, Chrome roundtrip and update behavior. Only then record the applicable acceptance evidence.

A local development certificate can support isolated sideload tests but is not public publisher trust. Never mark the existing release signing gate passed merely because this route was selected.

## Future direct-download channel
Keep packaging, signing and publishing as separate steps. Preserve stable Store identity and version progression for Store updates. A later direct EXE/MSI channel can use a trusted signing service without replacing the application source. Treat install coexistence, update ownership and data migration as explicit tests; switching signatures is not a drop-in migration for an installed MSIX.

## Official references
- Account types, new free onboarding and verification: https://learn.microsoft.com/en-us/windows/apps/publish/partner-center/open-a-developer-account
- Product reservation and submission: https://learn.microsoft.com/en-us/windows/apps/publish/get-started
- Identity, privacy policy and certification readiness: https://learn.microsoft.com/en-us/windows/apps/publish/store-readiness-checklist
- Registry/file visibility and restricted virtualization capability: https://learn.microsoft.com/en-us/windows/msix/desktop/flexible-virtualization
- Chrome Native Messaging registration: https://developer.chrome.com/docs/extensions/develop/concepts/native-messaging
- Store signing versus standalone EXE signing: https://learn.microsoft.com/en-us/windows/apps/package-and-deploy/code-signing-options
