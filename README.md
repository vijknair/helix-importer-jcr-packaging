# helix-importer-jcr-packaging

The `helix-importer-jcr-packaging` module provides APIs to help generate a JCR content package.  

## Output

The output of the createJcrPackage is a content package zip file, and a sidecar file that contains a map of image urls to the corresponding image 
file path in the content package.  This file then can be used by the [aem-import-helper](https://www.npmjs.com/package/@adobe/aem-import-helper) to 
automatically upload images to AEM.

## Usage

### Basic Usage (Franklin AEM)

```javascript
import { createJcrPackage, createPage } from '@adobe/helix-importer-jcr-packaging';

const pages = [
  createPage('/about', pageXml, 'https://example.com/about'),
  createPage('/contact', contactXml, 'https://example.com/contact')
];

await createJcrPackage(
  outputDirectory,
  pages,
  assetUrls,
  '/content/mysite',
  '/content/dam/mysite'
);
```

### Custom Empty Page Template (Non-Franklin AEM)

By default, `createJcrPackage` uses a Franklin-specific template for empty ancestor pages. If you're using a different AEM implementation, you can provide a custom empty page template:

```javascript
const customTemplate = `<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" jcr:primaryType="cq:Page">
  <jcr:content cq:template="/apps/myapp/templates/page" jcr:primaryType="cq:PageContent" sling:resourceType="myapp/components/page"/>
</jcr:root>`;

await createJcrPackage(
  outputDirectory,
  pages,
  assetUrls,
  '/content/mysite',
  '/content/dam/mysite',
  customTemplate  // Custom template for empty ancestor pages
);
```

**When to use a custom template:**
- Your AEM implementation uses custom page templates (not Franklin)
- You need different `cq:template` or `sling:resourceType` values
- You're migrating to a specific AEM project architecture

**Empty ancestor pages** are automatically created for parent paths that don't have explicit page content. For example, if you have a page at `/content/mysite/products/shoes/nike`, the following ancestor pages will be created using the empty page template:
- `/content`
- `/content/mysite`
- `/content/mysite/products`
- `/content/mysite/products/shoes`

## API Reference

### `createJcrPackage(outputDirectory, pages, assetUrls, siteContentPath, assetDamPath, [emptyPageTemplate])`

Creates a JCR content package from pages and assets.

**Parameters:**
- `outputDirectory` - The directory handle where the package will be written
- `pages` - Array of page objects created with `createPage()`
- `assetUrls` - Array of asset URLs found in the markdown
- `siteContentPath` - The path to the site content in AEM under `/content`
- `assetDamPath` - The path to the assets in AEM under `/content/dam`
- `emptyPageTemplate` (optional) - Custom XML template for empty ancestor pages. Defaults to Franklin template.

**Returns:** `Promise<void>`

### `createPage(path, data, url)`

Creates a page object for use with `createJcrPackage`.

**Parameters:**
- `path` - The page path
- `data` - The XML content of the page
- `url` - The source URL of the page

**Returns:** Page object
