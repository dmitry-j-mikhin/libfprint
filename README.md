> **This is a fork.** Upstream is [libfprint on freedesktop.org](https://gitlab.freedesktop.org/libfprint/libfprint).
> This repository carries the unofficial `elanmoc2` driver for ELAN "ARM-M4" match-on-chip
> fingerprint readers (04f3:0c00, 0c4c, 0c55, 0c5e, 0c7c, 0c90, …) plus a few fixes on top of it.
> See [About this fork](#about-this-fork) and [Related projects](#related-projects) below;
> the original README follows.

## About this fork

* Branch `elanmoc2`: Davide Depau's driver branch as-is, taken from
  [Depau/libfprint](https://gitlab.freedesktop.org/Depau/libfprint/-/tree/elanmoc2)
  (upstream merge request [!330](https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/330)),
  head `11f0316`.
* Branch `elanmoc2-fixes` (default): five commits on top of it, meant to be cherry-picked upstream:
  1. `elanmoc2: fix use-after-free when a verify/identify scan needs a retry` — fprintd crashed
     with SIGSEGV in `strlen()` after a `verify-remove-and-retry`; the retry `GError` was handed to
     libfprint without transferring ownership and the action was never completed.
     *The same bug is fixed independently in Depau/libfprint [!6](https://gitlab.freedesktop.org/Depau/libfprint/-/merge_requests/6) (`e50aaee6`).*
  2. `elanmoc2: report an unknown finger as no-match instead of an error` — `ELANMOC2_RESP_NOT_ENROLLED`
     is a regular no-match, not `FP_DEVICE_ERROR_DATA_NOT_FOUND`.
  3. `elanmoc2: do not time out while waiting for a finger` — identify/enroll waited for the finger with
     the generic 10 s USB timeout, so PAM authentication failed with "transfer timed out" unless the
     sensor was touched immediately. Only the cancellable finger-wait commands now wait without a timeout.
  4. `elanmoc2: add ELAN 04f3:0c55 (MSI CreatorPro M16 A12UJS)` — new device ID, also found in MSI
     Creator M16, Summit B14/B15, WF66/WF75/WF76 and WE76 according to linux-hardware.org.
  5. `hwdb: drop 04f3:0c7c and 04f3:0c90 from the unsupported list` — they were listed twice in the
     generated hwdb. *Also part of Depau/libfprint !6 (`5d5e1e88`).*
* Tested on Fedora 41 (libfprint 1.94.9, fprintd 1.94.5) with 04f3:0c55: enroll (8 stages, retry
  prompts), wrong finger → `verify-no-match`, light touch → `verify-remove-and-retry` followed by
  `verify-match`, sudo/PAM login. The driver still has no list/delete support (WIP upstream), so
  deleting a print in fprintd only removes the local file; re-enrolling the same finger replaces the
  template on the chip.
* Building: standard meson build (`meson setup build -Ddrivers=all && ninja -C build`). On Fedora the
  easiest way to install it is to rebuild the distribution SRPM with `git diff v1.94.9..elanmoc2-fixes`
  applied as a patch and pin the package with `dnf versionlock`.

## Related projects

The `elanmoc2` driver is not upstream yet, so many people carry their own copies. Known ones, as of
September 2026:

**Upstream / freedesktop.org GitLab**

| Where | What |
| --- | --- |
| [libfprint !330](https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/330) | The driver's merge request (Depau, open since 2023). Device-support issues are closed and redirected here and to the [Unsupported Devices wiki](https://gitlab.freedesktop.org/libfprint/wiki/-/wikis/Unsupported-Devices). |
| [Depau/libfprint !6](https://gitlab.freedesktop.org/Depau/libfprint/-/merge_requests/6) | Draft series by world.explorer.thomas (Aug 2026): retry-report ownership fix (same as commit 1 here) with a replay test, re-enrollment slot tracking, full-storage handling, enrollment state-machine completion, hwdb cleanup. Tested on 04f3:0c00. |
| [Depau/libfprint !3](https://gitlab.freedesktop.org/Depau/libfprint/-/merge_requests/3) | 04f3:0ca2 (LG laptops) support. |
| [Depau/libfprint !4](https://gitlab.freedesktop.org/Depau/libfprint/-/merge_requests/4) | Branch merged with current upstream master, indentation fix. |

**GitHub forks and packaging**

| Repository | What |
| --- | --- |
| [simon-ami/libfprint-elanmoc2-fork](https://github.com/simon-ami/libfprint-elanmoc2-fork) | Same finger-wait problem as commit 3, solved by raising the global receive timeout to 60 s, then 10 min. |
| [abdu-frh/Elan-0c4c-driver](https://github.com/abdu-frh/Elan-0c4c-driver) | Heavily reworked driver for 04f3:0c4c: list/unlock handling so fprintd stops deleting prints after failed verifies, USB reset on open, cancel handler, status-code mapping (0xFD as no-match). Diverged from Depau's branch. |
| [MN03SGO/libfprint-elanmoc2](https://github.com/MN03SGO/libfprint-elanmoc2) | 04f3:0c4d on Debian 13: no unconditional USB reset, real `clear_storage`, enrolled-count parsing fix. |
| [ITx-prash/libfprint-elan-04f3-0c6c](https://github.com/ITx-prash/libfprint-elan-04f3-0c6c), [nabil-hamisa/elan-0c6c-fingerprint-fix](https://github.com/nabil-hamisa/elan-0c6c-fingerprint-fix) | Patched driver files and installers for 04f3:0c6c. |
| [Debzillaa/libfprint-elanmoc2](https://github.com/Debzillaa/libfprint-elanmoc2) | 04f3:0ca0 patch with Debian packaging. |
| [Greek64/libfprint-elanmoc2-deb](https://github.com/Greek64/libfprint-elanmoc2-deb), [fschneid9/libfprint-elanmoc2-04f3-0c4c-ubuntu](https://github.com/fschneid9/libfprint-elanmoc2-04f3-0c4c-ubuntu) | Debian/Ubuntu package builds for 0c4c/0c00. |
| [skaldebane/libfprint-elanmoc2](https://github.com/skaldebane/libfprint-elanmoc2), [donCESAR12345/libfprint-elanmoc2](https://github.com/donCESAR12345/libfprint-elanmoc2), [theprashasst/elanmoc2-fedora-installer](https://github.com/theprashasst/elanmoc2-fedora-installer) | Fedora: COPR spec files and an installer script. COPR repos: `gaanee`, `johnc456`, `ajaygon`, `doncesar12345`/libfprint-elanmoc2. |
| [dje4321/elanmoc2](https://github.com/dje4321/elanmoc2), [sandptel/elanmoc2](https://github.com/sandptel/elanmoc2), [joenyr/libfprint-elanmoc2-working-0c00](https://github.com/joenyr/libfprint-elanmoc2-working-0c00), [deutereium/libfprintd-elanmoc2](https://github.com/deutereium/libfprintd-elanmoc2), [armiaab/libfprint](https://github.com/armiaab/libfprint), [VarunAgnihotri/libfprint-elanmoc2](https://github.com/VarunAgnihotri/libfprint-elanmoc2) | Copies of the branch, mostly for 04f3:0c00 / 0c4c, some rebased on newer libfprint. |
| [W34K3ST/libfprint-elanmoc2-mint-guide](https://github.com/W34K3ST/libfprint-elanmoc2-mint-guide), [jedbillyb/linux-fingerprint-drivers](https://github.com/jedbillyb/linux-fingerprint-drivers) | Install guide for Linux Mint; a catalogue of out-of-tree fingerprint drivers. |
| AUR: `libfprint-elanmoc2-git`, `libfprint-elanmoc2-working-git`, `libfprint-elanmoc2-newdrvs-git` | Arch packages building Depau's or geodic's branch. |

If you maintain another fork with fixes for this driver, please open an issue or PR here so it can be listed.

---



<div align="center">

# LibFPrint

*LibFPrint is part of the **[FPrint][Website]** project.*

<br/>

[![Button Website]][Website]
[![Button Documentation]][Documentation]

[![Button Supported]][Supported]
[![Button Unsupported]][Unsupported]

[![Button Contribute]][Contribute]
[![Button Contributors]][Contributors]

</div>

## History

**LibFPrint** was originally developed as part of an
academic project at the **[University Of Manchester]**.

It aimed to hide the differences between consumer
fingerprint scanners and provide a single uniform
API to application developers.

## Goal

The ultimate goal of the **FPrint** project is to make
fingerprint scanners widely and easily usable under
common Linux environments.

## License

`Section 6` of the license states that for compiled works that use
this library, such works must include **LibFPrint** copyright notices
alongside the copyright notices for the other parts of the work.

**LibFPrint** includes code from **NIST's** **[NBIS]** software distribution.

We include **Bozorth3** from the **[US Export Controlled]**
distribution, which we have determined to be fine
being shipped in an open source project.

## Get in *touch*

 - [IRC] - `#fprint` @ `irc.oftc.net`
 - [Matrix] - `#fprint:matrix.org` bridged to the IRC channel
 - [MailingList] - low traffic, not much used these days

<br/>

<div align="right">

[![Badge License]][License]

</div>


<!----------------------------------------------------------------------------->

[Documentation]: https://fprint.freedesktop.org/libfprint-dev/
[Contributors]: https://gitlab.freedesktop.org/libfprint/libfprint/-/graphs/master
[Unsupported]: https://gitlab.freedesktop.org/libfprint/wiki/-/wikis/Unsupported-Devices
[Supported]: https://fprint.freedesktop.org/supported-devices.html
[Website]: https://fprint.freedesktop.org/
[MailingList]: https://lists.freedesktop.org/mailman/listinfo/fprint
[IRC]: ircs://irc.oftc.net:6697/#fprint
[Matrix]: https://matrix.to/#/#fprint:matrix.org

[Contribute]: ./HACKING.md
[License]: ./COPYING

[University Of Manchester]: https://www.manchester.ac.uk/
[US Export Controlled]: https://fprint.freedesktop.org/us-export-control.html
[NBIS]: http://fingerprint.nist.gov/NBIS/index.html


<!---------------------------------[ Badges ]---------------------------------->

[Badge License]: https://img.shields.io/badge/License-LGPL2.1-015d93.svg?style=for-the-badge&labelColor=blue


<!---------------------------------[ Buttons ]--------------------------------->

[Button Documentation]: https://img.shields.io/badge/Documentation-04ACE6?style=for-the-badge&logoColor=white&logo=BookStack
[Button Contributors]: https://img.shields.io/badge/Contributors-FF4F8B?style=for-the-badge&logoColor=white&logo=ActiGraph
[Button Unsupported]: https://img.shields.io/badge/Unsupported_Devices-EF2D5E?style=for-the-badge&logoColor=white&logo=AdBlock
[Button Contribute]: https://img.shields.io/badge/Contribute-66459B?style=for-the-badge&logoColor=white&logo=Git
[Button Supported]: https://img.shields.io/badge/Supported_Devices-428813?style=for-the-badge&logoColor=white&logo=AdGuard
[Button Website]: https://img.shields.io/badge/Homepage-3B80AE?style=for-the-badge&logoColor=white&logo=freedesktopDotOrg
