---
name: persistence-cloudinary
description: Cloudinary integration for image upload, storage, and management
metadata:
  tags: cloudinary, image-upload, cloud-storage, media-management
---

# Cloudinary Image Storage

## Dependencies

```json
"cloudinary": "^2.0.0"
```

## Environment Configuration

Add to `.env`:

```bash
cloudinary_name=your_cloud_name
cloudinary_key=your_api_key
cloudinary_secret=your_api_secret
```

## Cloudinary Configuration

```javascript
// models/image-store.js
import { v2 as cloudinary } from "cloudinary";
import { writeFileSync, unlinkSync } from "fs";
import dotenv from "dotenv";

dotenv.config();

cloudinary.config({
  cloud_name: process.env.cloudinary_name,
  api_key: process.env.cloudinary_key,
  api_secret: process.env.cloudinary_secret,
});
```

## ImageStore Utility Class

```javascript
export const imageStore = {
  async uploadImage(imagefile) {
    // Save buffer to temporary file
    const tmpPath = `./tmp/${imagefile.hapi.filename}`;
    writeFileSync(tmpPath, imagefile._data);
    
    // Upload to Cloudinary
    const result = await cloudinary.uploader.upload(tmpPath);
    
    // Clean up temporary file
    unlinkSync(tmpPath);
    
    return result.url;
  },

  async getAllImages() {
    const result = await cloudinary.api.resources();
    return result.resources;
  },

  async deleteImage(publicId) {
    await cloudinary.uploader.destroy(publicId, {});
  }
};
```

## Database Schema with Image Field

```javascript
const collectionSchema = new Schema({
  title: String,
  img: String,
  userid: {
    type: Schema.Types.ObjectId,
    ref: "User",
  },
});
```

## Store Methods for Image Handling

```javascript
async updateCollection(updatedCollection) {
  const collection = await Collection.findOne({ _id: updatedCollection._id });
  collection.title = updatedCollection.title;
  collection.img = updatedCollection.img;
  await collection.save();
}
```

## Route Configuration

Enable multipart payload handling for file uploads:

```javascript
{
  method: "POST",
  path: "/collection/{id}/uploadimage",
  handler: collectionController.uploadImage,
  options: {
    payload: {
      multipart: true,
      output: "data",
      maxBytes: 209715200, // 200MB
      parse: true,
    },
  },
}
```

## Controller Handler

```javascript
async uploadImage(request, h) {
  try {
    const collection = await db.collectionStore.getCollectionById(request.params.id);
    const file = request.payload.imagefile;
    
    if (Object.keys(file).length > 0) {
      const url = await imageStore.uploadImage(file);
      collection.img = url;
      await db.collectionStore.updateCollection(collection);
    }
    
    return h.redirect(`/collection/${collection._id}`);
  } catch (err) {
    return h.view("main", { errors: [{ message: err.message }] });
  }
}
```

## View Layer - File Upload Component

```handlebars
<div class="card">
  <div class="card-content">
    <div class="content">
      <form action="/collection/{{collection._id}}/uploadimage" method="POST" enctype="multipart/form-data">
        <div class="file has-name is-fullwidth">
          <label class="file-label">
            <input class="file-input" type="file" name="imagefile" accept="image/*">
            <span class="file-cta">
              <span class="file-icon">
                <i class="fas fa-upload"></i>
              </span>
              <span class="file-label">
                Choose image…
              </span>
            </span>
          </label>
        </div>
        <button type="submit" class="button is-primary">Upload</button>
      </form>
    </div>
  </div>
</div>

{{#if collection.img}}
<div class="card">
  <div class="card-image">
    <figure class="image">
      <img src="{{collection.img}}" alt="Collection Image">
    </figure>
  </div>
</div>
{{/if}}
```

## Image Deletion

```javascript
async deleteImage(request, h) {
  const collection = await db.collectionStore.getCollectionById(request.params.id);
  
  if (collection.img) {
    // Extract public_id from Cloudinary URL
    const publicId = collection.img.split("/").pop().split(".")[0];
    await imageStore.deleteImage(publicId);
    
    collection.img = "";
    await db.collectionStore.updateCollection(collection);
  }
  
  return h.redirect(`/collection/${collection._id}`);
}
```

## Setup Steps

1. **Sign up**: Register at https://cloudinary.com/
2. **Get credentials**: Access dashboard to retrieve `cloud_name`, `api_key`, and `api_secret`
3. **Configure environment**: Add credentials to `.env` file
4. **Install package**: Run `npm install cloudinary`
5. **Create tmp directory**: Ensure `./tmp/` exists for temporary file storage
6. **Add image field**: Update collection schema to include `img` field
7. **Implement routes**: Add upload/delete routes with multipart configuration
8. **Create views**: Add file upload UI with Bulma components

## Notes

- The upload strategy saves in-memory buffer to temporary file before Cloudinary transmission
- Clean up temporary files after upload to avoid disk clutter
- Support file sizes up to 200MB (configurable via `maxBytes`)
- Use `accept="image/*"` attribute to restrict file picker to images only
- Store image URL in database, not the actual image data
- Extract `public_id` from Cloudinary URL when deleting images
