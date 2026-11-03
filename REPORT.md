### CivicConnect - Duplicate Complaint Detection and Statistics Fixes

**Problems Identified:**
- The Official Dashboard statistics were missing proper logic to accurately summarize counts, particularly missing fields for "High Priority Complaints" and "Department-wise Complaint Counts".
- The duplicate complaint logic was missing. Multiple citizens reporting the same issue in the same category were being treated as entirely unique, inflating the statistics and cluttering the admin view.
- The `get_admin_complaints` endpoint returned all duplicate complaints, making it harder to track genuine issues.

**Files Modified:**
- `backend_api.py`

**Changes Implemented:**
- **Schema Migration:** Updated `init_db` to include `parent_id`, `is_duplicate`, and `duplicate_count` to the `complaints` table using ALTER TABLE to preserve all existing data safely.
- **Duplicate Detection Logic:** In the `create_complaint` endpoint, implemented a check for existing active complaints with the exact same category and location. If found, the new complaint is flagged as a duplicate (`is_duplicate = 1`), mapped to the original (`parent_id = matched_id`), and the original's `duplicate_count` is incremented. Genuine citizen reports are not deleted.
- **Admin & Dashboard Statistics Updates:**
  - `get_admin_complaints` and `get_dashboard_statistics` now filter out complaints where `is_duplicate = 1` for their distinct queries.
  - Added new KPI fields to the response dictionaries: `duplicate_reports` (sum of `duplicate_count` across main complaints), `high_priority_complaints`, and `department_wise_counts`.

**Tests Executed and Results:**
- `python verify_apis.py`: Successfully passed all 15 API module verifications, including authentication, complaint creation, statistics retrieval, and analytics endpoints without regressions.
- Tested custom duplicate logic script showing correct linking and count aggregation logic in action.

**Remaining Issues:**
- Duplicate detection is currently based on exact category and location string matching. In a production environment with proper GIS data, this could be upgraded to use bounding-box proximity (PostGIS) or fuzzy title matching.
