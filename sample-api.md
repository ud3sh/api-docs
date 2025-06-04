### 1. Institutions

#### `GET /institutions`
* **Description:** Retrieves a list of institutions. Supports pagination.
* **Authentication:** Required.
* **Query Parameters:**
    * `page` (integer, optional): Page number for pagination.
    * `limit` (integer, optional): Number of items per page.
* **Response (200 OK):**
    ```json
    {
        "data": [
            {
                "id": 1,
                "ipedsId": "123456",
                "name": "Example University",
                "website": "https://exampleuniversity.edu",
                "active": true
            }
        ],
        "pagination": {
            "currentPage": 1,
            "totalPages": 1,
            "totalItems": 1,
            "limit": 10
        }
    }
    ```

### 2. Upload Batches

Endpoints for managing batches of transcript uploads.

#### `POST /upload-batches`
* **Description:** Creates a new upload batch. This is typically done before getting pre-signed URLs for individual files within the batch.
* **Authentication:** Required.
* **Request Body:**
    ```json
    {
        "batchName": "Spring 2025 Transfer Applicants Batch 1",
        "fileMetadata": [
            {
                "name": "fileName",
                "size": 3202,
                "type": "application/pdf",
                "key": "ui-generated-key"
            }
        ]
    }
    ```
* **Response (201 Created):**
    ```json
    {
        "id": 323234342342,
        "uploadUrls": [
            {
                "transcriptId": 11310012
                "key": "ui-generated-key",
                "uploadUrl": "https://edvsrly.s3.amazonaws.com/2013031/11310012?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20250604%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20250604T120000Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Signature=abcd1234ef567890example"
            }
        ]
    }
    ```

#### `GET /upload-batches`
* **Description:** Lists upload batches for an institution.
* **Authentication:** Required.
* **Query Parameters:** `page`, `limit`, `status`.
* **Response (200 OK):** (Paginated list of batch objects, where each object may contain fields like `id`, `batchName`, etc.)
---

### 3. Transcripts
Endpoints related to individual transcript uploads. This includes the crucial step of getting a pre-signed URL.

#### `GET /transcripts/{transcriptId}`
* **Description:** Retrieves details of a specific transcript upload.
* **Authentication:** Required.
* **Response (200 OK):**
    ```json
    {
        "id": 101,
        "batchId": 1,
        "uploadedByUserId": 2,
        "institutionId": 1,
        "originalFileName": "student_transcript.pdf",
        "fileType": "application/pdf",
        "fileSizeBytes": 102400,
        "status": "parsed",
        "fidelityScore": 0.85,
        "processedTranscript": { /* standardized transcript json format */  },
        "evaluationNotes": null
    }
    ```

#### `PUT /transcripts/{transcriptId}`
* **Description:** Updates a transcript upload. This could be used by users to add notes or by the system (e.g., `transcript-processor` indirectly, or a subsequent service) to update status, processed data, etc. The `processedTranscript` field would contain the standardized transcript data.
* **Authentication:** Required.
* **Request Body (Example for user adding notes):**
    ```json
    {
        "evaluationNotes": "Student has strong grades in relevant math courses."
    }
    ```
* **Request Body (Example for system updating after processing):**
    ```json
    {
        "processedTranscript": { /* standardized transcript json format */ }
    }
    ```
* **Response (200 OK):** (The updated transcript upload object)

---

### 4. Courses

Endpoints for managing course definitions, primarily for the evaluating institution.

#### `POST /courses`
* **Description:** Creates a new course definition for an institution.
* **Authentication:** Required.
* **Request Body:**
    ```json
    {
        "courseCode": "CS101",
        "courseTitle": "Introduction to Computer Science",
        "description": "Fundamentals of programming and computer science.",
        "credits": 3.00,
        "department": "Computer Science"
    }
    ```
* **Response (201 Created):** (The created course object)

#### `GET /courses`
* **Description:** Lists courses for an institution.
* **Authentication:** Required.
* **Query Parameters:** `page`, `limit`, `courseCode`, `department`.
* **Response (200 OK):** (Paginated list of course objects)

#### `GET /courses/{courseId}`
* **Description:** Retrieves a specific course by its ID.
* **Authentication:** Required.
* **Response (200 OK):** (Course object)

#### `PUT /courses/{courseId}`
* **Description:** Updates an existing course.
* **Authentication:** Required.
* **Request Body:** (Fields to update, e.g., `courseTitle`, `description`)
* **Response (200 OK):** (The updated course object)

---

### 5. Course Equivalencies

Endpoints for managing course equivalency rules.

#### `POST /course-equivalencies`
* **Description:** Creates a new course equivalency rule. The `sourceCourseId` refers to the ID of the course from the external institution (which should exist in a shared or referenceable courses table, identified by `sourceInstitutionId`). The `equivalentInstitutionCourseId` refers to the ID of the course at the evaluating institution.
* **Authentication:** Required.
* **Request Body:**
    ```json
    {
        "sourceInstitutionId": 2, // ID of the external institution
        "sourceCourseId": 213142,    // ID of the course from the external institution
        "equivalentInstitutionCourseId": 9012300, // ID of the course at the evaluating institution
        "mappingType": "direct",
        "creditsAwarded": 3.00,
        "notes": "Direct transfer for Intro to CS."
    }
    ```
* **Response (201 Created):** (The created course equivalency object)

#### `GET /course-equivalencies`
* **Description:** Lists course equivalencies for an evaluating institution.
* **Authentication:** Required.
* **Query Parameters:** `page`, `limit`, `sourceInstitutionId`, `mappingType`.
* **Response (200 OK):** (Paginated list of equivalency objects)

---

### 6. Scoring Rules

Endpoints for defining and managing scoring rules.

#### `POST /scoring-rules`
* **Description:** Creates a new scoring rule. The `criteria` field will contain details like `minGpa` and `requiredCourses`.
* **Authentication:** Required.
* **Request Body:**
    ```json
    {
        "ruleName": "Standard STEM Applicant Rule",
        "description": "Rule for evaluating STEM applicants based on GPA and key courses.",
        "criteria": {
            "minGpa": 3.2,
            "requiredCourses": ["MATH201", "PHYS101"],
            "courseGradeWeights": { "A": 4, "B": 3 }
        }
    }
    ```
* **Response (201 Created):** (The created scoring rule object)

---

## Note on example standardized transcript
```json
{
  "studentInformation": {
    "firstName": "Jane",
    "middleName": "Marie",
    "lastName": "Doe",
    "studentIdAtSource": "SID789001",
    "dateOfBirth": "2003-07-15",
    "contactEmail": "jane.doe@example.com", // If available
    "contactPhone": "+1-555-0100" // If available
  },
  "issuingInstitution": {
    "name": "Central State University",
    "city": "Metropolis",
    "state": "IL",
    "country": "USA",
    "accreditation": "Regionally Accredited by HLC" // If available
  },
  "academicTerms": [
    {
      "termId": "term_2022_fall",
      "termName": "Fall Semester 2022",
      "startDate": "2022-08-29",
      "endDate": "2022-12-16",
      "courses": [
        {
          "courseCode": "ENG101",
          "courseTitle": "Composition I",
          "creditsAttempted": 3.0,
          "creditsEarned": 3.0,
          "gradeReceived": "A-",
          "gradePoints": 11.1, // (e.g., 3.7 grade point * 3 credits)
          "notes": "Transfer equivalent: WRIT100"
        },
        {
          "courseCode": "MATH203",
          "courseTitle": "Calculus I",
          "creditsAttempted": 4.0,
          "creditsEarned": 4.0,
          "gradeReceived": "B+",
          "gradePoints": 13.2 // (e.g., 3.3 grade point * 4 credits)
        }
      ],
      "termGpa": 3.471, // (total grade points / total credits attempted for term)
      "termCreditsAttempted": 7.0,
      "termCreditsEarned": 7.0,
      "academicStatus": "Good Standing" // e.g., Dean's List, Probation
    }
    // ... more terms
  ],
  "degreeInformation": [
    {
      "degreeTitle": "Associate of Arts",
      "major": "General Studies",
      "conferralDate": "2023-05-15",
      "honors": "cum laude"
    }
  ],
  "cumulativeSummary": {
    "gpa": 3.65,
    "totalCreditsAttempted": 60.0,
    "totalCreditsEarned": 57.0,
    "classRank": null // If available
  },
  "transcriptLegend": [ // Key-value pairs from the transcript's grading scale
    {"grade": "A", "description": "Excellent", "qualityPoints": 4.0},
    {"grade": "A-", "description": "Very Good", "qualityPoints": 3.7}
    // ... other grade mappings
  ],
  "additionalInformation": { // For any unstructured but potentially relevant info
    "notes": "Student completed an international exchange program in Spring 2023.",
    "disciplinaryActions": [] // If mentioned
  },
  "rawExtractedText": "Full text extracted from OCR or text-based PDF for auditing..." // Optional, but useful for verification and error correction
}
