# Store work through the Blanxer MCP

The site's code only reads Blanxer. Everything you change in the store while building, you change through the Blanxer MCP. Its tools act as the signed-in person, with that person's role.

How to connect:
- **Claude Code:** comes with this plugin, or `claude mcp add --transport http blanxer https://mcp.blanxer.com/mcp`.
- **Cursor / Codex / claude.ai:** add `https://mcp.blanxer.com/mcp` as a remote MCP server.

The merchant signs in to Blanxer in the browser. You never see a key.

| While building the site you… | MCP tool | The site reads it from |
|---|---|---|
| Check SEO gaps | `blanxer_audit_seo` | — |
| Write product titles and descriptions for search | `blanxer_update_product_status_seo`, `blanxer_bulk_update_products` | product SEO fields → page metadata |
| Write category and brand SEO | `blanxer_bulk_update_categories_seo`, `blanxer_update_category`, `blanxer_update_brand` | `check_domain.categories`, brand details |
| Verify Google Search Console, add GA/GTM | `blanxer_set_site_verification` | store plugins → `verificationMetadata` |
| Add products the merchant describes or sends photos of | `blanxer_create_product`, `blanxer_bulk_create_products`, `blanxer_bulk_import_products`, `blanxer_upload_product_image`, `blanxer_request_image_upload` | product endpoints |
| Create categories and brands for the menus | `blanxer_create_category`, `blanxer_create_brand`, `blanxer_reorder_categories` | categories / brands |
| Set up launch coupons | `blanxer_create_coupon` | coupons |
| Fix delivery places, charges and COD | `blanxer_get_delivery_charges`, `blanxer_update_delivery_charges` | delivery table |
| Change name, logo, favicon or colours (shared with the Blanxer site) | `blanxer_update_store_profile`, `blanxer_update_appearance` | `check_domain.customization` |
| Write blog posts | `blanxer_create_blog_post`, `blanxer_set_blog_post_status` | blog |
| **Save the website link** | `blanxer_set_storefront_url` | — |
| Get the feed and sitemap links | `blanxer_get_store_context` | — |
| Retire the old subdomain site | `blanxer_update_appearance { store_visibility: 'hidden_on_subdomain' }` | — |

Don't use these on a custom site, because they change the Blanxer-hosted site:
- `blanxer_add_section`
- `blanxer_update_section`
- `blanxer_apply_page_template`
- navigation and footer editors

Confirm before bulk changes (rule 13).
