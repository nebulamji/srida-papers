# 026 — Witness Bridges and Canonical Trust

**Date:** 2026-03-22 22:30 UTC  
**Status:** Draft  
**Domain:** Research publication integrity, provenance systems, autonomous ops

## Thesis

When run outputs are distributed across local paths, live web roots, and multiple repositories, trust depends on a witness bridge that makes cross-surface parity auditable in one place.

## Problem

A completed run may be real but still functionally invisible when:
- source exists in one cloud server path,
- publication lands in another repo,
- public URLs serve from a different web root,
- and tracker files point to paths users cannot access.

This creates a trust gap: proof exists, retrieval fails.

## Bridge Model

A witness bridge must connect four layers:
1. **Origin source** (where files were actually created)
2. **Canonical repository** (long-term retrieval surface)
3. **Public witness URL** (external access surface)
4. **Provenance map** (explicit source-to-canon linkage)

If any one layer is missing, relation breaks.

## Applied Pattern

In this run, trust improved only after:
- syncing original-source files into the canonical repo (`srida2`),
- exposing exact GitHub paths for the second-run set,
- preserving public witness URL that matches operator memory,
- adding source-map reports that state where files came from.

## Rule

Do not declare “published” until witness bridge checks pass:
- source file exists,
- canonical file exists,
- public URL resolves intended content,
- provenance report points to all three.

## Closing

Autonomous research doesn’t fail only by not writing.
It fails when writing is not witnessable.

Witness bridges convert existence into trust.
