# Sprint 3 Task 3 - Accessibility in Cybersecurity Report

## Introduction

Desert Oil’s security controls protect the staff who can use them and miss the ones who cannot. A login page NVDA cannot read, or an awareness video without captions, leaves those staff exposed to the attacks the controls exist to stop.

This report covers Task 3 of Sprint 3 (Human Factors) of the CyBlack internship. The brief asks for three things:

1.  Research accessibility in cybersecurity and why it matters.
2.  Report on that research, the challenges found and the solutions to them.
3.  Run an accessibility test on the team’s reports and presentations, including a test with assistive technology.

The work uses Desert Oil, a fictional oil and gas company modelled on Saudi Aramco, as the organisation. Desert Oil is also the company behind the Sprint 3 Task 2 awareness campaign built in VIVIDA Studio, so the test covers the campaign as well as the written reports.

In this report, accessibility means that people with visual, hearing, motor and cognitive disabilities can use the same information, products and services as everyone else. Accessibility testing checks digital content against that goal.

## Overview

I tested five deliverables against WCAG 2.2 Level AA with one automated checker per format, a manual check, and the NVDA screen reader on the case study PDF. The automated checkers miss problems of meaning, so the manual and NVDA checks carry as much weight as the scans.

### Deliverables tested

| **No.** | **Deliverable** | **Format** | **Why it was chosen** |
| --- | --- | --- | --- |
| D1 | Task 1 case study (Desert Oil, Shamoon 2012) | Word and PDF | A written report, named in the brief |
| D2 | Task 2 VIVIDA campaign report | Word and PDF | A written report, named in the brief |
| D3 | Sprint 3 presentation | PowerPoint and PDF | A presentation, named in the brief |
| D4 | VIVIDA image post (sample) | Image on the VIVIDA web platform | Content staff actually see; checks alt text and contrast |
| D5 | VIVIDA video reel (sample) | Video on the VIVIDA web platform | Content staff actually see; checks captions and audio |

###   

### Tools

| **Tool** | **Cost** | **Used on** | **What it checks** |
| --- | --- | --- | --- |
| Microsoft Accessibility Checker (Review tab) | Built into Word and PowerPoint | D1, D2, D3 | Alt text, headings, table headers, reading order, contrast |
| PAVE | Free, web | PDF version of D1 | PDF tags, title, language, reading order, headings, tables |
| Fezzant Accessibility Scanner | Free Chrome extension | D4, D5 | WCAG issues on the live web page |
| Chrome DevTools (Inspect) | Built into Chrome | D4 | The image’s actual alt text in the page code |
| WCAG contrast formula and Colour Contrast Analyser | Free | Brand colours | Text and background contrast ratio |
| NVDA | Free screen reader for Windows | D1 PDF | How the content sounds to a blind user |

### Method

1.  Run the automated checker for the file format and record every error and warning.
2.  Check by hand what tools cannot judge: whether alt text is meaningful, whether link text makes sense out of context, whether colour alone carries meaning.
3.  Read the whole file with NVDA, keyboard only, and note anything confusing or skipped.
4.  Fix the issues, re-run the tests, and record the before and after result.  
    

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-001.png)

\[Diagram: test workflow, from automated scan to manual check to NVDA pass to fix and retest\]

## Findings

### Part 1: Research on accessibility in cybersecurity

The UK National Cyber Security Centre treats accessibility as a security requirement. Staff who cannot use a control work around it, and each workaround opens a weakness the control was meant to close.

#### Why it matters

-   **Scale.** About 1.3 billion people, 1 in 6 worldwide, live with a significant disability ([WHO, 2023](https://www.who.int/News-Room/Fact-Sheets/Detail/Disability-and-Health)). At Desert Oil’s size, that means thousands of staff and contractors.
-   **Workarounds create risk.** Staff bypass security that gets in their way, so the NCSC asks organisations to make the security itself accessible ([CSO Online on the NCSC blog, 2023](https://www.csoonline.com/article/575381/accessibility-should-be-a-cybersecurity-priority-says-uk-ncsc.html)). A blind employee might share a password with a sighted colleague, or switch off a protection that blocks the screen reader.
-   **Unclear controls give false confidence.** Mislabelled buttons, audio-only or visual-only warnings and red/green status colours can lead a person to believe a control is on when it is off (NCSC, as above).
-   **Training has to reach everyone.** A deaf employee learns nothing from an uncaptioned awareness video. The Sprint 3 VIVIDA reel has this gap.
-   **Disability can be temporary or situational.** The NCSC describes three colleagues: one deaf, one with an ear infection, one in a noisy room without headphones. Captions designed for the deaf colleague also serve the other two.
-   **Inclusion keeps access control intact.** Accessible design lets authorised users reach their tools and leaves the rules on who is authorised unchanged ([Accessibility in Cybersecurity: A Practical, Living Guide](https://accessiblesecurity.org)).

#### Where security shuts people out

| **Security control** | **Who is blocked** | **Resulting risk** | **Accessible solution** |
| --- | --- | --- | --- |
| Visual CAPTCHA | Blind and low-vision users | Locked out, or asks someone else to log in for them | Bot detection that needs no puzzle; object-recognition tests only where unavoidable |
| One-time code that must be retyped | Users with memory, dyslexia or motor impairments | Failed logins, help desk resets, weaker fallback methods | Allow paste and autofill of codes, as WCAG 2.2 criterion 3.3.8 requires ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication.html)) |
| Password-only login | Users with cognitive impairments | Weak or reused passwords written down | Allow password managers; offer passkeys, hardware keys or biometrics |
| Single MFA method (for example SMS only) | Users who cannot use that one channel | No safe way to log in | Offer several MFA methods; NCSC notes this also adds resilience |
| Session timeout with no warning | Screen reader and switch users, who work slower | Lost work, pressure to disable timeouts | Warn before timeout and allow an extension |
| Red/green alerts and dashboards | Colour-blind users, including analysts | Missed or misread alerts | Pair colour with text and icons |
| Awareness videos and posters | Deaf, hard of hearing, low-vision users | Staff never receive the training | Captions, transcripts, alt text, high contrast |

#### Standards that apply

-   **WCAG 2.2** is the main web accessibility standard. Most organisations target Level AA.
-   **Criterion 3.3.8 Accessible Authentication (Minimum), Level AA**, bars logins that depend on a memory or puzzle test unless the site offers help, such as a password manager or copy and paste ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication.html)). WCAG does not count hardware keys, biometrics or device PINs as such tests.
-   The NCSC advises asking vendors for an accessibility statement or WCAG compliance before buying security tools.

### Part 2: Accessibility testing report

Each step below is one test. Record the result in the table for that deliverable, then add the screenshot where the placeholder sits.

#### Step 1. Install NVDA

Download NVDA from nvaccess.org and install it on the Windows 11 PC. Start it with Ctrl + Alt + N and stop it with Insert + Q.

Keys used in every NVDA pass:

| **Key** | **What it does** |
| --- | --- |
| Insert + Down Arrow | Read everything from the cursor |
| Ctrl | Stop speech |
| H | Jump to the next heading |
| G | Jump to the next image |
| T | Jump to the next table |
| K | Jump to the next link |
| Insert + F7 | List all headings and links |

#### Step 2. Run the Word Accessibility Checker on D1 and D2

Open the file in Word, go to Review, then Check Accessibility. Record every error and warning, then fix them in the pane.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-002.png)D2 VIVIDA report, Word Accessibility Assistant showing 47 missing alt text, before fixes  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-003.png)

D2 VIVIDA report, Word Accessibility Assistant showing Looks good, no issues found.  
  
|![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-004.png)

D1 case study, Word Accessibility Assistant showing Looks good, no issues found.

#### Step 3. Run the PowerPoint Accessibility Checker on D3

Open the deck, go to Review, then Check Accessibility. Also, open Home, Arrange, Selection Pane to check the reading order on each slide, since the screen reader reads from the bottom of that list upward.

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-005.png)  
  
D3 presentation, Accessibility Assistant showing 63 missing alt text, 10 missing slide titles and 5 reading order issues.  

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-006.png)

D3 presentation, Accessibility Assistant showing Looks good, no issues found

#### Step 4. Export accessible PDFs

In Word or PowerPoint, choose File, Save As, PDF, then Options, and tick Document structure tags for accessibility. Without the tags, a screen reader sees the PDF as one block of text.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-007.png)

Save As PDF options with the tags box ticked

#### Step 5. Check the PDFs with PAVE

Upload each PDF to PAVE (pave-pdf.org), review the reported issues, fix what it can fix (title, language, tags), and download the corrected file.

  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-008.png)

PAVE Step 1, page 1 table tagged as Formula, before the fix.  
  
  
  
  
  
  
  
  
  
  

  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-009.png)

PAVE Step 2, page 1 reading order scrambled (table first, title third).  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-010.png)PAVE Step 2, page 1 reading order fixed (1 to 9 top to bottom).  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-011.png)

PAVE Step 3, heading structure with one H1 and nine H2.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-012.png)

PAVE Step 4, threats table with First Row set as header.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-013.png)

PAVE Step 8, title, author and language set, no warnings.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-014.png)

PAVE Network Error when uploading the 41-page VIVIDA report.

#### Step 6. Scan the VIVIDA pages with Fezzant and check alt text in DevTools (D4, D5)

Open the image post and the reel in Chrome. Run the Fezzant Accessibility Scanner from the extensions menu. Export the results as a PDF.  
  
Then right-click the image, choose Inspect and read its alt attribute, because scanners only check that alt text exists, not what it says.

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-015.png)  
Fezzant scan results for D4

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-016.png)

Chrome DevTools showing the graphic’s alt text, alt=“Edit the first reference image”.  
  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-017.png)Fezzant scan results for the D5 Fahad phishing reel page

#### Step 7. Check colour contrast

Use the WebAIM Contrast Checker on the main text and background colours, including the Desert Oil dark blue and dark green brand colours. Normal text needs a ratio of at least 4.5 to 1; large text needs 3 to 1.

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-018.png)  
contrast check result for the main brand colour pair

#### Step 8. Check captions on the reel (D5)

Play the reel with the sound off. Every spoken word and important sound should appear as a caption. If the platform has no caption option, write a transcript to sit beside the reel.  
  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-019.png)

D5 reel playing with the sound off, showing no captions.

#### Step 9. NVDA pass on every deliverable

Using the keyboard only, read each file with NVDA. Note where it skips content, reads in the wrong order, says only “graphic” or a file name for an image, or reads a table without headers.  
  
![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-020.png)

NVDA Speech Viewer, H key moving through all 10 headings of the D1 PDF, ending “no next heading”.  

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-021.png)

NVDA Speech Viewer reading the three D1 tables (8 rows by 3 columns, 7 by 2, 7 by 2).  

![](./Sprint3-Task3-Accessibility-Report-Ukah-Paul_images/image-022.png)

NVDA Speech Viewer moving across table cells, announcing column numbers but not header names\]

#### Step 10. Fix and retest

Fix every issue found, re-run Steps 2 to 9 on the fixed files, and fill in the After column below.

#### Results

| **Check** | **Tool** | **D1 Case study** | **D2 VIVIDA report** | **D3 Presentation** | **D4 Image post** | **D5 Reel** |
| --- | --- | --- | --- | --- | --- | --- |
| Every image has meaningful alt text | Office checker, Fezzant, DevTools, NVDA | Pass | Fail (47) then Pass | Fail (63) then fixed: 9 images described, rest marked decorative | Fail (alt is the prompt title “Edit the first reference image”) | n/a (video; see captions) |
| Real heading styles used | Office checker, PAVE, NVDA (H key) | Pass (NVDA H key: 1 H1, 9 H2) | Pass | Fail (10) then fixed: each slide heading set as its title | n/a | n/a |
| Table header rows marked | Office checker, PAVE, NVDA (T key) | Word: Pass. PDF: Partial (tables read, column names not announced) | Pass | n/a (no tables) | n/a | n/a |
| Logical reading order | Reading Order Pane, PAVE, NVDA | Fail then Pass (PDF) | Not tested | Fail (5) then fixed: title first, content in visual order | Not tested | Not tested |
| Contrast at least 4.5 to 1 | Office checker, Fezzant, WCAG formula | Pass (Office checker) | Pass (Office checker) | Pass (Office checker) | Fail (Fezzant: 1 critical, VIVIDA page) | Fail (Fezzant: 14 elements, VIVIDA page) |
| Document title and language set | Office checker, PAVE | Pass (title, author, English) | Not tested | Not tested | n/a | n/a |
| PDF tagged | PAVE | Pass after PAVE fixes | Not tested in PAVE (upload failed: Network Error) | n/a (not exported) | n/a | n/a |
| Captions present | Manual | n/a | n/a | n/a | n/a | Fail (no captions; VIVIDA cannot add them) |
| Keyboard operable | Manual, NVDA | n/a | n/a | n/a | Not tested | Not tested |
| Automated tool errors, before then after | All | 0 in Word; 3 tagging errors fixed in PAVE | 47 then 0 | 78 then 0 | Fezzant: 5 (1 critical, 4 moderate) | Fezzant: 5 issue types |

n/a marks a check that does not apply to that format; Not tested marks a check outside this round. Colour-only signals and link text were not tested in this round and are left for the next test cycle.

#### Brand colour contrast

Text on any Desert Oil campaign colour passes WCAG AA (4.5 to 1 for normal text) and AAA (7 to 1). One pair fails: dark blue text on dark green, so the team should never combine those two.

| **Text colour** | **Background** | **Contrast ratio** | **AA (4.5:1)** | **AAA (7:1)** |
| --- | --- | --- | --- | --- |
| Charcoal #1F2933 | Warm white #FAF7F0 | 13.79 : 1 | Pass | Pass |
| White #FFFFFF | Dark blue #123558 | 12.53 : 1 | Pass | Pass |
| Warm white #FAF7F0 | Dark blue #123558 | 11.71 : 1 | Pass | Pass |
| White #FFFFFF | Dark green #1B4332 | 11.08 : 1 | Pass | Pass |
| Warm white #FAF7F0 | Dark green #1B4332 | 10.35 : 1 | Pass | Pass |
| Dark blue #123558 | Dark green #1B4332 | 1.13 : 1 | Fail | Fail |

I calculated the ratios with the WCAG 2.2 relative luminance formula, using the hex values from the campaign style guide.

#### Issues found and fixes

| **No.** | **Deliverable** | **Issue** | **WCAG criterion** | **Fix applied** | **Retest result** |
| --- | --- | --- | --- | --- | --- |
| 1 | D2 VIVIDA report | 47 images and screenshots had no alt text | 1.1.1 Non-text Content | Added a description to each image, or marked it decorative | Pass: Word reports no issues |
| 2 | D1 case study PDF | Decorative bottom border of the page 1 table was tagged as a Figure | 1.1.1 Non-text Content | Retagged as Artifact in PAVE so screen readers skip it | Fixed in PAVE |
| 3 | D1 case study PDF | Page 1 threats table was tagged as a Formula, so a screen reader would not read it as rows and columns | 1.3.1 Info and Relationships | Retagged as Table in PAVE; header row set in the Tables step | Pass: tagged as Table, First Row set as header cells, page 2 part joined to the same table |
| 4 | D1 case study PDF | Page 1 reading order was scrambled: the table was read first and the title third | 1.3.2 Meaningful Sequence | Used Detect Reading Order in PAVE, then checked it runs top to bottom (title, headings, paragraphs, table) | Pass: order 1 to 9 now matches the page |
| 5 | D1 case study PDF | Word’s repeated header rows are read by NVDA as data rows: “row 7, page 2, Threat” and “row 6, page 3, Stakeholder” | 1.3.1 Info and Relationships | Not fixed in PAVE; fix at source by keeping each table on one page or turning off Repeat as header row | Open: minor, recorded as a limitation |
| 6 | D1 case study PDF | Header rows are tagged, but NVDA in Acrobat Reader gives only the column number when moving across cells, not the header name (“column 2, A targeted fake email”) | 1.3.1 Info and Relationships | Turned on NVDA Row/column headers setting and retested; no change. Likely cause: header cells have no column scope set | Partial: tables read with rows and columns; column names not announced. Recommend checking with Acrobat Pro or PAC 2024 |
| 7 | D3 Presentation | 63 shapes and images had no alt text, including full-slide backgrounds, card shapes, lines and loose bullet and arrow text boxes | 1.1.1 Non-text Content | Wrote descriptions for the 9 meaningful images (logo, VIVIDA world screenshot, six campaign images, character style guide); marked the rest decorative | Pass: PowerPoint reports no issues |
| 8 | D3 Presentation | All 10 slides had no slide title; headings were plain text boxes | 2.4.2 Page Titled; 2.4.6 Headings and Labels | Set each slide’s visible heading as its slide title; all 10 titles unique | Pass: PowerPoint reports no issues |
| 9 | D3 Presentation | Reading order flagged on 5 slides; screen readers would hit backgrounds and stray bullets before the content | 1.3.2 Meaningful Sequence | Title placed first, content in visual order, decorative shapes skipped | Pass: PowerPoint reports no issues |
| 10 | D4 VIVIDA graphic page (“The Aftermath”) | Critical: 5 pieces of small grey text on the Studio page fail contrast: the Workspace and World labels, the Full prompt button, the creator name and the date | 1.4.3 Contrast (Minimum) | None possible: VIVIDA platform styling, outside the team’s control | Open: platform limitation, to report to VIVIDA |
| 11 | D4 VIVIDA graphic page | Moderate: heading levels skip (the “Prompt” label is an h3 with no h2 before it) | 1.3.1 Info and Relationships | None possible: platform code | Open: platform limitation, to report to VIVIDA |
| 12 | D4 VIVIDA graphic page | Moderate: page landmarks are broken: one main region sits inside another, the page has two main regions, and two landmarks share the same name | 1.3.1 Info and Relationships (best practice) | None possible: platform code | Open: platform limitation, to report to VIVIDA |
| 13 | D4 “The Aftermath” graphic | The image’s alt text is the AI prompt title, alt=“Edit the first reference image”, so a screen reader user hears an instruction, not the content or the words “The Aftermath” and “One click. Three costs.” Automated scanners passed it because alt text exists | 1.1.1 Non-text Content | Checked the Studio: VIVIDA sets alt text from the asset’s prompt title and has no option to edit alt text or rename the asset. Wherever the graphic is posted, use: “The Aftermath: three panels showing the costs of one click. 1, operational downtime at HQ; 2, infrastructure damage to a pipeline; 3, reputational damage. Caption: One click. Three costs.” | Open: platform limitation, to report to VIVIDA; fixed wherever the team posts the graphic |
| 14 | D5 Fahad phishing reel page | Same 5 platform issue types as the graphic page; contrast now fails on 14 pieces of small grey text (labels such as Characters, Locations, Duration, Provider, Created by, the date and the World label) | 1.4.3; 1.3.1 | None possible: platform code | Open: platform limitation, to report to VIVIDA |
| 15 | D5 Fahad phishing reel | The reel’s audio has no captions, so deaf and hard-of-hearing staff, or anyone watching with the sound off, miss the spoken content | 1.2.2 Captions (Prerecorded) | None possible in VIVIDA, which has no caption option. Outside it: caption the downloaded reel (a caption file in PowerPoint or Teams, or Clipchamp) or post a short transcript with it | Open: platform limitation, to report to VIVIDA |

####   
Challenges and solutions

These are the challenges met during testing and how each was handled.

| **Challenge** | **Solution** |
| --- | --- |
| Fezzant only scans web pages, not Word, PowerPoint or PDF files | Used the Office Accessibility Checkers for the source files and PAVE for the PDF |
| Automated tools passed problems a person would notice: VIVIDA’s alt text exists but is the AI prompt title, and the PDF table headers are tagged but not announced | Finished with a manual code check and an NVDA pass; tools check that something exists, not that it makes sense |
| Learning NVDA keystrokes takes time | Used the key table in Step 1 and Speech Viewer to see what NVDA says |
| VIVIDA cannot edit alt text or add captions, and its own pages fail contrast and structure checks | Recorded as platform limitations, added fixes outside VIVIDA (descriptions and captions after download), and recommended that VIVIDA publish a WCAG 2.2 AA statement |
| PAVE’s automatic detection mis-tagged content in the case study PDF: a table as a Formula, a border line as a Figure, and a scrambled reading order on page 1 | Checked every region by hand, retagged them, and re-ran Detect Reading Order; automated tagging always needs a human review |
| Changing regions in PAVE Step 1 reset the reading order, and later steps stayed locked until it was redone | Make all region changes first, then set the reading order once on every page |
| Heading levels and table header cells could not be set in Step 1, where it seemed they belonged | Found them in the later steps: Step 3 Organize Heading Levels and Step 4 Define Table Structure (First Row) |
| Word’s repeated table header rows on pages 2 and 3 appear in the PDF as ordinary data rows, so a screen reader reads the header text twice | Noted as a limitation; at source, keep short tables on one page so no header repeats |
| NVDA keyboard shortcuts did not work at first on a laptop: focus was on the Acrobat toolbar, and F7 is shared with a media key | Clicked inside the page before testing, set Caps Lock as the NVDA key, and used the H key to move through headings |
| During continuous reading, NVDA seemed to split numbered headings into “1.” and the title, which looked like a broken heading | Checked with the H key: each heading is announced once, for example “1. Top human factor threats in oil and gas, heading level 2”, so the structure is correct |
| PAVE returned “Network Error” when uploading the 41-page VIVIDA report PDF, which holds dozens of screenshots | PAVE testing was limited to the 4-page case study; the VIVIDA report was checked with the Word Accessibility Checker (47 issues fixed to 0) instead |

### Part 3: Lessons learned

1.  **Staff work around controls they cannot use.** The workaround, such as a shared password, is weaker than the control it replaces.
2.  **Automated tools need a human check.** Fezzant passed the VIVIDA graphic because it had alt text; the page code showed that text was the AI prompt title. NVDA showed that the PDF table headers PAVE had tagged were never announced.
3.  **Build accessibility in while writing.** Applying heading styles, alt text and table headers as you write takes less time than fixing 78 issues in a finished deck.
4.  **Campaign content needs alt text and captions.** VIVIDA provides neither, so the Desert Oil team must add them after download or the campaign misses deaf and blind staff.
5.  **Offer more than one login method.** Several MFA options help disabled staff and give everyone a fallback when one method fails.
6.  **The checklist applies to every future deliverable.** The brief requires all future team documents, content and presentations to be accessible and tested, so the team will run the Part 2 steps on each one.

### Part 4: Recommendations

#### For VIVIDA (the platform vendor)

1.  Add an editable alt text field to every image and video, and stop using the AI prompt title as alt text (currently, for example, “Edit the first reference image”).
2.  Add a way to create and edit captions on video reels.
3.  Fix the Studio pages: raise the contrast of the small grey labels, correct the heading order, and keep a single main landmark per page.
4.  Publish a WCAG 2.2 AA accessibility statement and a roadmap for the fixes above.

#### For Desert Oil (before any campaign asset goes out)

1.  Add an accessibility step to the publishing checklist: every downloaded VIVIDA image gets a real description wherever it is posted (Teams, email, intranet, slides).
2.  Caption every reel after download, using a caption file in PowerPoint or Teams or a video editor such as Clipchamp, or post a short transcript with it.
3.  Ask every security and training vendor for a WCAG 2.2 AA statement before purchase, as the NCSC advises.
4.  Use only the brand colour pairs that pass contrast; never place dark blue text on dark green.

#### For the team’s own documents

1.  Build accessibility in while writing: heading styles, alt text, table header rows, real slide titles.
2.  Run the Office Accessibility Checker before every export.
3.  Export PDFs with document structure tags, then check them in PAVE, or PAC 2024 for large files.
4.  Keep tables on one page where possible so Word does not repeat header rows that screen readers read as data.
5.  Finish every deliverable with a short NVDA pass (headings with H, tables with T, reading order with read-all); automated tools miss meaning, as the VIVIDA alt text showed.
6.  State test results accurately in presentations (for example, the tools used and the issues found) rather than “no major issues”.

## Glossary

| **Term** | **Meaning** |
| --- | --- |
| Accessibility | Making content and services usable by people of all abilities |
| Alt text | A short written description of an image, read aloud by screen readers |
| Assistive technology | Hardware or software that helps disabled people use a computer, such as a screen reader |
| CAPTCHA | A test meant to tell humans from bots, often a distorted image or puzzle |
| Contrast ratio | How different the text colour is from its background, from 1 to 1 up to 21 to 1 |
| MFA | Multi-factor authentication: logging in with two or more kinds of proof |
| NVDA | NonVisual Desktop Access, a free screen reader for Windows |
| PAVE | A free web tool that checks and repairs PDF accessibility |
| Reading order | The sequence in which a screen reader reads the items on a page or slide |
| Screen reader | Software that reads on-screen content aloud for blind and low-vision users |
| Tagged PDF | A PDF that carries structure (headings, tables, images) so assistive technology can read it |
| WCAG | Web Content Accessibility Guidelines, the W3C standard for accessible digital content; Level AA is the common target |

## Conclusion

I tested the team’s five Sprint 3 deliverables and fixed 128 accessibility issues: 47 missing image descriptions in the VIVIDA report, 78 issues in the presentation (alt text, slide titles and reading order) and 3 tagging errors in the case study PDF. Every Word and PowerPoint file now passes the Office Accessibility Checker, and NVDA reads the case study PDF with one H1, nine H2s and three real tables.

The screen reader test mattered most. Automated tools passed two problems that NVDA and a manual code check exposed: PDF table headers that are tagged but not announced, and VIVIDA alt text that is only the AI prompt title. VIVIDA also cannot edit alt text or add captions, and its own pages fail contrast and structure checks; these stay open as platform limitations to report to the vendor.

For Desert Oil, the next steps are the Part 4 recommendations: an accessibility step in campaign publishing, captions on every reel after download, WCAG 2.2 AA statements from security and training vendors, and an NVDA check on every team deliverable before it is shared.

## References

-   World Health Organization, [Disability and health fact sheet](https://www.who.int/News-Room/Fact-Sheets/Detail/Disability-and-Health), 7 March 2023
-   CSO Online, [Accessibility should be a cybersecurity priority, says UK NCSC](https://www.csoonline.com/article/575381/accessibility-should-be-a-cybersecurity-priority-says-uk-ncsc.html), 19 May 2023
-   W3C, [Understanding SC 3.3.8: Accessible Authentication (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication.html)
-   Merhoff and Yisa, [Accessibility in Cybersecurity: A Practical, Living Guide](https://accessiblesecurity.org)
-   NCSC, [Accessibility as a cyber security priority](https://www.ncsc.gov.uk/blog-post/accessibility-as-a-cyber-security-priority) (link from the task brief)