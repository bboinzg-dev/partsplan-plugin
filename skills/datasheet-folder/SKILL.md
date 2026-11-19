---
name: datasheet-folder
description: Fill a hardware design project's datasheet folder with manufacturer datasheet PDFs from PartsPlan — one check_bom call for the BOM, then one download per part saved as datasheets/<part number>.pdf. Use with KiCad, EasyEDA/JLCPCB, Altium, Eagle/Fusion, OrCAD or code-based design projects (atopile, tscircuit) when the project keeps datasheets next to the design, when PDFs are missing for some parts, when another tool or skill expects local datasheet PDFs (for example the kicad-happy datasheet skills), or when no distributor API key is set up.
---

# Datasheet PDFs for a design project folder

Many circuit design projects keep the datasheets of their parts in a folder next to the design, and several review and analysis tools read the PDFs from there (the kicad-happy skill bundle, for example, extracts facts from whatever PDFs are in `<project>/datasheets/`). This skill fills that folder from the PartsPlan archive. It only places files; extraction, analysis and design changes stay with the other tools and the user. It is not part of any of those tools and is not affiliated with them.

## Where the parts come from
Each line needs two facts: the manufacturer and the manufacturer part number. Read the project's own names by meaning, not by exact spelling.
- KiCad: symbol fields in the schematic, or an exported BOM. The part number field goes by many names (`MPN`, `Manufacturer Part Number`, `Manufacturer P/N`, `Mfr P/N`); the maker is usually `Manufacturer` or `Mfr`.
- EasyEDA / JLCPCB BOM: `Manufacturer Part` is the part number, `Manufacturer` the maker, `Supplier Part` the JLCPCB part code (a C number).
- Altium: `Manufacturer 1` and `Manufacturer Part Number 1` in a BOM export (higher numbers are alternates); projects that renamed them use names such as `MPN 1`.
- Eagle or Fusion, OrCAD, and code-based flows such as atopile or tscircuit (where the part number is an attribute in the source, for example `manufacturerPartNumber`): the same two facts under the tool's own names.
- A JLCPCB part code goes into `codes.easyeda_jlcpcb` of that line. It helps when the maker is written the way that listing writes it; it does not replace the manufacturer and does not pick a document.
- Lines that carry only a value (10k, 100n) and no part number are left out.

## Where the files go
Use the datasheet folder the project already has (for example `datasheets/`, `Datasheets/` or `docs/datasheets/`). If there is none, create `datasheets/` next to the project file. Do not change schematic, board or library files. If the user asks to link the files from the design, a KiCad symbol's Datasheet field can hold `${KIPRJMOD}/datasheets/<name>.pdf` or the document's `document_url`.

## Steps
1. List the parts as above and keep one line per manufacturer and part number. Leave out lines that have no part number and lines whose PDF is already in the folder (the file name is given in step 3). A line with a part number but no manufacturer is still sent; it comes back `brand_missing`.
2. Call `check_bom` with the lines (`brand`, `mpn`, optional `ref` and `codes`). One call takes up to 100 lines; send a longer BOM in several calls.
3. For each line with `status: "found"`, download its `pdf_url` with a plain GET (no extra headers, follow redirects) and save it as `<folder>/<name>.pdf`. `<name>` is the part number as written in the project, with spaces and the characters `/ \ : * ? " < > | , ;` replaced by `_`. Run at most two downloads at a time.
4. Check each file: its SHA-256 equals `documents[0].doc_sha256` of that line (the download also sends it in the `x-doc-sha256` header). If it does not match, or the download failed, delete the file and count the line as not placed.
5. Tell the user:
   - the files placed, each with `documents[0].document_url`;
   - the lines not placed and why, including the lines left out in step 1;
   - the `note` of a placed line when it has one (for example a family or combined datasheet, where the table for the exact part still has to be confirmed on the page);
   - that an archived copy may not be the manufacturer's current edition.

## The download link
`pdf_url` is signed: it works once and expires 15 minutes after `check_bom` returned it, so download right after the call. A refused download returns a small JSON body with `status` and `message`; the message says whether to retry with the same link or to ask for a new one (call `check_bom` again for those lines, once). `api_quota_exceeded` means the account's call limit is used up: stop and report which lines are left.

## Lines that are not `found`
- `found_variant`: the archive holds that manufacturer's document for an ordering variant, not for this exact part number. Report it with the variant named in `similar_part_numbers`; do not save it under the requested part number unless the user decides to.
- `other_brand_only`: only another manufacturer's document exists for that part number. That is a different part; do not place it.
- `no_document`: not in the archive, which does not mean the part has no datasheet. The manufacturer's site may have it, and `request_datasheet_collection` records a collection request if the user wants one.
- `brand_missing`, `invalid`: fix the BOM line first; do not guess the manufacturer.

## Limits
Each download counts as 10 calls toward the account's call limit; the free plan's daily limit covers about ten downloads. `limits` in the `check_bom` response is the remaining BOM-line budget, not the call budget. For a BOM that will not fit, place the ICs and other parts with pin, timing or thermal questions first and say which lines are left. Do not retry in a loop when a limit is reported.

## What the files are
- Each PDF is the archived original of the manufacturer's datasheet; the SHA-256 of the file is the document id.
- An archived copy may not be the manufacturer's current edition. `find_datasheet` lists the revision markers printed in each archived copy, and several archived editions are listed separately; `check_bom` links the first document of the line.
- PartsPlan does not validate circuits and does not choose parts.
