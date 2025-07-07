# my-api-docs

1. Expert Routes
GET /api/admin/audit-logs/export
 Purpose:
Export audit logs in JSON or CSV format with filters for category, severity, date range, and search terms.
 Access:
Authentication required
Authorization:
oRole must be admin
oadminLevel must be >= 10

2. Markets
GET /api/admin/markets/:id
  Purpose:
Fetch market details by MongoDB _id.
 Permissions Required: markets.read
 Route Param:
id: string – Market ID

3. Markets Update
PUT /api/admin/markets/:id
 Purpose:
Update a market’s data (partial updates supported).
 Permissions Required: markets.update
 Request Body (Zod validated):
ts
CopyEdit
{
  name?: string,
  slug?: string,         // Lowercase, alphanumeric, dash-separated
  description?: string,
  metaTitle?: string,
  metaDescription?: string,
  keywords?: string[],   // Array of strings
  iconPath?: string,
  sortOrder?: number,
  isActive?: boolean
}
 Slug Conflict Check:
If slug is changed:
It must be unique
If duplicate exists, API returns an error

4. Delete Market
DELETE /api/admin/markets/:id
   Purpose:
Delete a market if it has no associated subcategories.
 Permissions Required: markets.delete
   Condition:
If any subcategory exists under this market, deletion is blocked.

5. Permissions
GET /api/admin/permissions
  Location: src/app/api/admin/permissions/route.ts
  Purpose:
Returns system-defined permissions used for role-based access control (RBAC).
  Access Control:
Auth required via auth()
Must have users.read permission
Unauthorized users receive 401 or 403
  Input: None

6. SubCategory by ID
GET /api/admin/subcategories/:id
  Purpose:
Fetch subcategory details, related market, and latest 10 topics.
  Returns:
Subcategory info
Market info (name, slug)
10 latest topics (name, slug, createdAt)

PUT /api/admin/subcategories/:id
 Purpose:
Update subcategory fields.
 Validations:
Zod schema
Unique slug within market
Referenced market must exist
  Returns:
Updated subcategory with market info

DELETE /api/admin/subcategories/:id
🔹 Purpose:
Delete subcategory only if it has no associated topics.
📌 Checks:
Subcategory must exist
Block if topics are still linked
📤 Returns:
success: true if deleted
Proper error otherwise
🧪 Sample Response (GET):
json
CopyEdit
{
  "id": "64fa12b90a7e0eaf89e3f211",
  "name": "Indian Agriculture",
  "slug": "indian-agriculture",
  "marketId": "64e5bd7a8451b267fca171ae",
  "market": {
    "_id": "64e5bd7a8451b267fca171ae",
    "name": "Farming Market",
    "slug": "farming-market"
  },
  "topics": [
    {
      "_id": "64fa12b90a7e0eaf89e3f210",
      "name": "Crop Rotation",
      "slug": "crop-rotation",
      "createdAt": "2024-05-12T08:00:00.000Z"
    }
  ]
}

7. Topics & SubCategories List
GET /api/admin/subcategories
 Purpose:
Fetch all subcategories with filtering, pagination, sorting.

GET /api/admin/topics/:id
 Purpose:
Fetch detailed topic info with subcategory and market hierarchy.

DELETE /api/admin/topics/:id
 Purpose:
Delete a topic if no DataVisualizations are linked.
 Conditions:
Topic must exist
No linked DataVisualization records
 Authorization:
Role: admin
Must be authenticated
 Success Response:
HTTP 200

Charts Upload & Management
 Base Endpoint: /api/admin/upload/charts
 POST /upload/charts
 Purpose:
Upload up to 5 chart images. Validates type, quota, stores in S3, and generates thumbnails.
 Roles: admin, editor, analyst
 Form Fields:
files[]: up to 5 images
relatedContentId: Optional ObjectId
relatedContentType: datavisualization, report, user_upload
generateThumbnails: Optional (default true)

 GET /upload/charts
 Purpose:
List uploaded chart files with filters and pagination.
 Query Parameters:
page (default: 1)
limit (default: 20)
status: uploading, processing, completed, failed
userId: optional
relatedContentId: optional
 Returns:
Chart file list with metadata
Pagination info

 DELETE /upload/charts?fileId=...
 Purpose:
Delete chart file from S3 and DB.
 Access:
Admin or original uploader
 Query:
fileId (required)
 Behavior:
Deletes original & thumbnail from S3
Removes DB record
Access validated

Data Upload & Processing
 Endpoint: /api/admin/upload/data
 POST /upload/data
 Purpose:
Upload 1 CSV/Excel file → validate, parse, store, generate chart suggestions.
 Roles: admin, editor, analyst
 Form Fields:
file: Required
relatedContentId: Optional
relatedContentType: datavisualization (default), report, user_upload
 Rules:
Only 1 file
Valid CSV/Excel
Must follow structure
Checked against storage quota
 Returns:
File metadata
Presigned S3 URL
Parsed data summary (rows, columns, types, nulls)
Chart suggestions with confidence
First 10 preview rows
Data quality: completeness, consistency, validity, uniqueness

 GET /upload/data
 Purpose:
List uploaded/processed data files.
 Query Params:
page: default 1
limit: default 20
status: uploading, processing, completed, failed
userId: optional
relatedContentId: optional
 Returns:
Uploaded file list
Metadata + chart suggestions
Pagination

🗑 DELETE /upload/data?fileId=...
 Purpose:
Delete data file from S3 and DB
 Access:
Admin or file owner
 Query Param:
fileId: required
 Behavior:
Verify permission
Delete from S3
Remove DB record
