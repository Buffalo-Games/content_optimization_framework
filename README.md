# BHC AI Content Framework Tool

Internal tool for Buffalo Games / EastPoint Sports marketing and digital teams. This tool implements the 12-dimension AI content framework, covering Syndigo attribute mapping, Amazon-first workflow, product type intake wizard, field validator, and a market-specific search term word bank. Designed to make content creation consistent, faster, and keyword-optimized across the full BHC catalog — from puzzles to rec sports.

## What's in this repo

- `index.html` — the full standalone tool (no build step, no dependencies, opens in any browser)
- `keywords.json` — search term data file, updated periodically from Helium10 / BigQuery exports
- `DEPLOY.md` — deployment and data connection guide

## Deployment

**SharePoint:** Upload `index.html` to a SharePoint document library. Embed via an Embed web part using the SharePoint file URL.

**GitHub Pages:** Live at **https://buffalo-games.github.io/content_optimization_framework/**

**Local:** Open `index.html` directly in any browser. No server required.

## Updating the tool

Edit `index.html` and push to main. If GitHub Pages is enabled, changes go live automatically within ~60 seconds.

## Updating keyword data

Edit `keywords.json` with new terms from Helium10 or BigQuery exports. Set `vol` fields to actual search volume numbers when available. Push to main. The tool fetches this file on load — no HTML changes needed.

## Data connection (future state)

See `DEPLOY.md` for instructions on connecting live BigQuery data via Google Apps Script or Cloud Run. The tool is architected to swap `keywords.json` for a live API endpoint with one line of config change.

## Internal use only

This tool is for BHC internal use. Do not share the repo URL externally without approval.
