<!-- START_METADATA
---
title: PSP API changelog
sidebar_label: Changelog
sidebar_position: 200
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Changelog

All notable changes to the current API will be documented in this file.
To learn about API versioning, see
[Common topics: API Lifecycle](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/api-lifecycle/).

## October 2020

Launched PSP API v3:

Vipps launched the PSP API v3 to coincide with our migration of our users' cards from PAN to EMVco network
tokens. This technological migration is Vipps' strategy for achieving delegated
SCA once PSD2 comes into effect for card payments on January 1, 2021.

The PSP API v3 is functionally identical to PSP API v2 apart from the
additional payment source format. There are also minor changes to naming of
properties to bring the API in line with Vipps' API standards.

Encrypted cards will still be sent for our users cards that have not yet been
migrated. Please see our [migration guide](v2-deprecation.md).
