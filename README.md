# [BigCommerce](https://www.bigcommerce.com/) imgix integrational guide

## Disclaimer:

This guide presents a working solution for integrating imgix into your Bigcommerce site. All configurations are made to work on top of [Bigcommerce's](https://www.bigcommerce.com/) front end and CDN, but will have certain limitations that are detailed in our [Caveats \& Warnings](#caveats--warnings) section. We cannot, unfortunately, guarantee expected results if at any time Bigcommerce were to make changes to its system in ways that create breaking issues with this guide.

This guide is written for BigCommerce users with Stencil experience (BigCommerce's theme engine), and will require you to write, modify, and test custom code.

# Table of contents

- [BigCommerce imgix integration](#bigcommerce-imgix-integration)
  - [Disclaimer:](#disclaimer)
  - [This guide presents a working solution for integrating imgix into your Bigcommerce site. All configurations are made to work on top of Bigcommerce's front end and CDN, but will have certain limitations that are detailed in our caveats \& warnings section below. We cannot, unfortunately, guarantee expected results if at any time Bigcommerce were to make changes to its system in ways that create breaking issues with this guide.](#this-guide-presents-a-working-solution-for-integrating-imgix-into-your-bigcommerce-site-all-configurations-are-made-to-work-on-top-of-bigcommerces-front-end-and-cdn-but-will-have-certain-limitations-that-are-detailed-in-our-caveats--warnings-section-below-we-cannot-unfortunately-guarantee-expected-results-if-at-any-time-bigcommerce-were-to-make-changes-to-its-system-in-ways-that-create-breaking-issues-with-this-guide)
- [Table of contents](#table-of-contents)
- [Install the imgix.js script in your BigCommerce settings](#install-the-imgixjs-script-in-your-bigcommerce-settings)
- [Editing your theme files](#editing-your-theme-files)
  - [Editing your theme files locally (recommended)](#editing-your-theme-files-locally-recommended)
  - [Editing your theme files in BigCommerce's UI.](#editing-your-theme-files-in-bigcommerces-ui)
- [Editing images in template files](#editing-images-in-template-files)
  - [Replacing `getImageSrcset`](#replacing-getimagesrcset)
  - [Replacing images in `data-*` attributes](#replacing-images-in-data--attributes)
- [Common template files to edit](#common-template-files-to-edit)
  - [`responsive-img.html`](#responsive-imghtml)
- [Editing the `product-view.html` file](#editing-the-product-viewhtml-file)
  - [Caveats \& Warnings](#caveats--warnings)
- [Examples](#examples)

# Install the imgix.js script in your BigCommerce settings

1. Open the BigCommerce admin UI
2. Click on **Storefront** in the left sidebar
3. Click **Create a Script**
    1. Name the script (`imgix-js`)
    2. Put it in the **Head**
    3. Use in **All Pages**
    4. Use the **Essential** category
    5. Script type: `URL`
        1. Grab the latest version from cdnjs to use: https://cdnjs.com/libraries/imgix.js
    6. Load method: `default`
4. Save and move to the next section below

# Editing your theme files

You can edit your BigCommerce theme files locally, or in the BigCommerce UI.

We strongly recommend editing your theme files locally, as that is the most efficient way to preview and test changes to your BigCommerce themes.

## Editing your theme files locally (recommended)

Editing locally makes it easier to preview your changes with the [Stencil CLI](https://developer.bigcommerce.com/stencil-docs/installing-stencil-cli/installing-stencil).

1. In your BigCommerce admin, navigate to **Storefront** in your admin sidepanel
2. Go to **Themes** in your BigCommerce store control panel
3. In your theme, click **Advanced** → **Download Current Theme**

From there, generate a [Stencil CLI Token](https://developer.bigcommerce.com/stencil-docs/installing-stencil-cli/live-previewing-a-theme#obtaining-store-api-credentials) and install the [Stencil CLI](https://developer.bigcommerce.com/stencil-docs/installing-stencil-cli/installing-stencil) to begin editing your theme files locally.

## Editing your theme files in BigCommerce's UI.

If editing theme files locally is daunting, you can edit your theme files directly in BigCommerce. This is typically not recommended because editing the theme in the BigCommerce IU is a time-consuming process because since the site is rebuilt every time a changed is saved.

1. In your BigCommerce admin, navigate to **Storefront** in your admin sidepanel
2. Go to **Themes** in your BigCommerce store control panel
3. Find your theme and click **make a copy**
    1. **This step is critical. Do not skip this step, otherwise you can break your site**
4. Activate the theme copy
5. In your newly activated theme, click **Advanced** → **Edit theme code**

# Editing images in template files

Once you have the template code available to edit, you must locate code where images are being output/modified in your theme code. This will vary from theme to theme, and you may even have plugins that modify how images are output.

To modify the image string outputs, we are going to be using these [handlebar helpers provided by BigCommerce](https://developer.bigcommerce.com/stencil-docs/reference-docs/handlebars-helpers-reference):

- `stripQuerystring`: Strips the cache query parameters from BigCommerce (ex: `?c=1`)
- `strReplace`: Replaces the BigCommerce hostname in the image URL paths with the imgix hostname
- `getImage`: Gets the image URL. To get the Origin Image, you will likely use `getImage image` in your template code

BigCommerce uses the handlebar helpers to get images from the image object. Examples:

- `getImage this`
- `getImage product.main_image`

This outputs the BigCommerce image URL. Use `strReplace` to replace the BigCommerce domain with your own imgix domain. Examples:

- `stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')`
- `stripQuerystring (strReplace (getImage product.main_image) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')`

## Replacing `getImageSrcset`

Additionally, BigCommerce uses `getImageSrcset` to generate a `srcset` of images. You should not use `getImgaeSrcset` because it generates multiple variations of image paths (`/640w/image.png`, `/740w/image.png`, etc), which will dramatically increase your Origin Image counts.

In general, you will replace `getImageSrcset` either a custom `srcset`:

```
srcset="{{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=1000&auto=compress,format 1000w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=500&auto=compress,format 500w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=250&auto=compress,format 250w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=100&auto=compress,format 100w"
```

... or you will delete it and let imgix handle the `srcset` generation with imgix.js's `ix-src` attribute:

```
ix-src="{{stripQuerystring (strReplace (getImage image) 'cdn11.bigcommerce.com/s-YOUR_BIGCOMMERCE_STORE_ID' 'YOUR_IMGIX_SUBDOMAIN.imgix.net')}}?auto=compress,format"
```

## Replacing images in `data-*` attributes

Some template files use image srcsets in `data-*` attributes (ex: `data-image-gallery-new-image-srcset`). You must replace the `srcset` with manually generated srcset. Example:

```
data-image-gallery-new-image-srcset="{{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=1000&auto=compress,format 1000w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=500&auto=compress,format 500w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=250&auto=compress,format 250w, {{stripQuerystring (strReplace (getImage this) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?w=100&auto=compress,format 100w"
```

# Common template files to edit

Most images are output and modified in the following theme files:

- `/templates/components/common/responsive-img.html`: Outputs the `<img>` tags with `srcset` attributes
- `/templates/components/products/product-view.html`: Passes image URLs to `responsive-img.html`
- `/templates/components/carousel.html`: Outputs an image in the carousel element

Depending on your theme, you may have more images to modify, but these are the most common files to edit for BigCommerce templates.

## `responsive-img.html`

This file commonly controls how most responsive images are displayed in BigCommerce themes.

It's referenced in other theme files like so:

```
                {{> components/common/responsive-img
                    image=product.main_image
                    class="productView-image--default"
                    fallback_size=theme_settings.product_size
                    lazyload=theme_settings.lazyload_mode
                    default_image=theme_settings.default_image_product
                    otherAttributes="data-main-image"
                }}
```

**Steps to edit this file:**

1. Navigate to this file: `/templates/components/common/responsive-img.html`
2. Replace all `src` values with something like this:
      `{{stripQuerystring (strReplace (getImage image) 'cdn11.bigcommerce.com/s-YOUR_BIGCOMMERCE_STORE_ID' 'YOUR_IMGIX_SUBDOMAIN.imgix.net')}}?auto=compress,format`
3. Replace the first occurrence of `srcset` with this:
            `ix-src="{{stripQuerystring (strReplace (getImage image) 'cdn11.bigcommerce.com/s-YOUR_BIGCOMMERCE_STORE_ID' 'YOUR_IMGIX_SUBDOMAIN.imgix.net')}}?auto=compress,format"`
4. Delete all `srcset` values. For example, delete this:

```
srcset="{{getImageSrcset image 1x=(default lqip_size '80w')}}"
```

Use this HTML file as an example for how your `responsive-img.html`  template should look:

# Editing the `product-view.html` file

This file commonly controls how images are displayed in a product page.

Note that there are several image handlers in the product page template. Ex:

- `getImageSrcset this`
- `getImageSrcset product.main_image`

You must use the right image handler when modifying the file.

Here is a good template to use for replacing BigCommerce image URLs with imgix URLs:

```
{{stripQuerystring (strReplace (getImage IMAGE_HANDLER) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?auto=compress,format
```

In cases where a `width` and `height` is specified in the HTML element, you should modify the image so that it fits the `width` and `height` values - example:

```
{{stripQuerystring (strReplace (getImage IMAGE_HANDLER) 'cdn11.bigcommerce.com/s-STORE_ID' 'SUBDOMAIN.imgix.net')}}?auto=compress,format&w=608&h=608&fit=crop
```

## Caveats & Warnings

**Updating custom themes**

Note that integrating imgix into a BigCommerce theme involves modifying your theme code.

**[BigCommerce cannot push updates to custom or third-party themes.](https://support.bigcommerce.com/s/article/Marketplace-Theme-Updates)** If you need to update a custom theme while keeping imgix functionality, you will:

1. Need to download a new version of the BigCommerce theme
2. Reapply your imgix edits to that new theme version

**Plugin interference**

If you have any plugins that control the output of your images (such as a gallery plugin), you may need to either disable/modify that theme's plugin files to be compatible with imgix, or you may need to modify that plugin's functionality.

# Examples

You can find example files in the [/examples](/examples) folder of this repo.
