# Color-Picker-Chrome-Extension
- Developed a Chrome extension that enables users to easily pick colors from any web page.
- Implemented the extension using HTML, CSS, and JavaScript, utilizing Chrome Extension APIs.
- Designed an intuitive user interface with a color picker tool and real-time color preview.
- Integrated functionality to copy the selected color code to the clipboard.
```


const ROWS_TOTAL = 37; // generate dummy rows count (example)
const PAGE_SIZE = 10;

// helper to format timestamp -> "12 Sep, 2025"
function formatDate(timestampMs) {
  if (!timestampMs) return "";
  const d = new Date(Number(timestampMs));
  const opts = { day: "2-digit", month: "short", year: "numeric" };
  // month short gives "Sep", we want "12 Sep, 2025"
  const formatter = new Intl.DateTimeFormat("en-GB", opts);
  const parts = formatter.format(d); // e.g., "12 Sep 2025"
  // Insert comma after month
  const [day, month, year] = parts.split(" ");
  return `${day} ${month}, ${year}`;
}

// generate dummy data with timestamp (ms)
function generateDummyRows(count) {
  const rows = [];
  const now = Date.now();
  for (let i = 1; i <= count; i++) {
    // serial-style IDs
    const workitemId = `WI-${1000 + i}`; // e.g., WI-1001
    const loanId = `LN-${2000 + i}`; // e.g., LN-2001
    const userId = `user_${String(i).padStart(3, "0")}`;
    const applicant = ["John Doe", "Alice Smith", "Rahul Kumar", "Priya Verma", "Arjun Patel"][(i - 1) % 5];
    // createdAt: spread recent days (ms)
    const createdAt = now - i * 86400000; // i days ago
    rows.push({
      workitemId,
      loanId,
      userId,
      applicant,
      status: "PENDING",
      createdAt, // timestamp in ms
    });
  }
  return rows;
}

export default function CheckerQueueComponent() {
  const allRows = useMemo(() => generateDummyRows(ROWS_TOTAL), []);
  const [query, setQuery] = useState("");
  const [page, setPage] = useState(1);

  // filter rows by search across specified fields
  const filteredRows = useMemo(() => {
    const q = (query || "").trim().toLowerCase();
    if (!q) return allRows;
    return allRows.filter((r) => {
      // search workitem id, loan id, user id, applicant, formatted date
      if (String(r.workitemId).toLowerCase().includes(q)) return true;
      if (String(r.loanId).toLowerCase().includes(q)) return true;
      if (String(r.userId).toLowerCase().includes(q)) return true;
      if (String(r.applicant).toLowerCase().includes(q)) return true;
      if (formatDate(r.createdAt).toLowerCase().includes(q)) return true;
      return false;
    });
  }, [allRows, query]);

  // pagination
  const totalPages = Math.max(1, Math.ceil(filteredRows.length / PAGE_SIZE));
  const currentPage = Math.min(Math.max(1, page), totalPages);
  const pageStart = (currentPage - 1) * PAGE_SIZE;
  const pageRows = filteredRows.slice(pageStart, pageStart + PAGE_SIZE);

  const handleView = (row) => {
    // placeholder - replace with navigation / modal as needed
    console.log("VIEW clicked:", row.workitemId);
    alert(`Open details for ${row.workitemId}`);
  };

  const goPrev = () => setPage((p) => Math.max(1, p - 1));
  const goNext = () => setPage((p) => Math.min(totalPages, p + 1));

  // reset page to 1 when query changes
  React.useEffect(() => {
    setPage(1);
  }, [query]);

  return (
    <div className="checker-page">
      <div className="container-fluid p-4">
        <div className="card checker-card">
          <div className="card-body">
            {/* header: title left, search right */}
            <div className="d-flex align-items-center justify-content-between mb-3">
              <h4 className="cq-heading mb-0">CHECKER QUEUE</h4>

              <div style={{ minWidth: 320 }}>
                <input
                  type="search"
                  className="form-control cq-search-input"
                  placeholder="Search workitem / loan / user / applicant / date"
                  value={query}
                  onChange={(e) => setQuery(e.target.value)}
                  aria-label="Search checker queue"
                />
              </div>
            </div>

            {/* table */}
            <div className="table-responsive">
              <table className="table cq-table mb-0">
                <thead>
                  <tr>
                    <th className="text-uppercase">WORKITEM ID</th>
                    <th className="text-uppercase">LOAN ID</th>
                    <th className="text-uppercase">USER ID</th>
                    <th className="text-uppercase">APPLICANT NAME</th>
                    <th className="text-uppercase">STATUS</th>
                    <th className="text-uppercase">CREATED</th>
                    <th className="text-uppercase">ACTION</th>
                  </tr>
                </thead>

                <tbody>
                  {pageRows.length === 0 ? (
                    <tr>
                      <td colSpan="7" className="text-center py-4 text-muted">
                        No records found.
                      </td>
                    </tr>
                  ) : (
                    pageRows.map((r) => (
                      <tr key={r.workitemId} className="cq-row">
                        <td className="fw-bold">{r.workitemId}</td>
                        <td className="fw-bold">{r.loanId}</td>
                        <td>{r.userId}</td>
                        <td>{r.applicant}</td>
                        <td>
                          <span className="badge bg-warning text-dark">PENDING</span>
                        </td>
                        <td>{formatDate(r.createdAt)}</td>
                        <td>
                          <button className="btn btn-sm btn-primary" onClick={() => handleView(r)}>
                            VIEW
                          </button>
                        </td>
                      </tr>
                    ))
                  )}
                </tbody>
              </table>
            </div>

            {/* pagination */}
            <div className="d-flex justify-content-end align-items-center mt-3 gap-2">
              <div className="text-muted me-3">
                Showing {pageRows.length} of {filteredRows.length}
              </div>
              <nav aria-label="Page navigation">
                <ul className="pagination pagination-sm mb-0">
                  <li className={`page-item ${currentPage === 1 ? "disabled" : ""}`}>
                    <button className="page-link" onClick={goPrev}>
                      Previous
                    </button>
                  </li>
                  <li className="page-item disabled">
                    <span className="page-link">
                      {currentPage} / {totalPages}
                    </span>
                  </li>
                  <li className={`page-item ${currentPage === totalPages ? "disabled" : ""}`}>
                    <button className="page-link" onClick={goNext}>
                      Next
                    </button>
                  </li>
                </ul>
              </nav>
            </div>
            {/* end card-body */}
          </div>
        </div>
      </div>
    </div>
  );
}
```

```
/* CheckerQueueComponent.css */

/* page background (full-screen subtle) */
body, html, #root {
  height: 100%;
}

/* center card and give page gradient */
.checker-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #0b3b78 0%, #113a66 100%);
  padding: 28px 20px;
}

/* card that holds the table (white) */
.checker-card {
  border-radius: 12px;
  background: #ffffff;
  box-shadow: 0 8px 30px rgba(3, 20, 60, 0.12);
  border: none;
}

/* heading */
.cq-heading {
  color: #0b3b78;
  font-weight: 700;
  margin: 0;
}

/* search input custom */
.cq-search-input {
  border: 2px solid #0b3b78;
  color: #0b3b78;
  padding: 6px 10px;
  border-radius: 6px;
}

/* table rounded corners */
.cq-table {
  border-collapse: separate;
  border-spacing: 0;
  width: 100%;
  border-radius: 8px;
  overflow: hidden;
}

/* header style (uppercase set by class already) */
.cq-table thead th {
  background: linear-gradient(90deg, #003a7b 0%, #004a9b 100%);
  color: #fff;
  font-weight: 700;
  padding: 12px 14px;
  border: none;
  text-transform: uppercase;
  font-size: 13px;
}

/* body cells */
.cq-table tbody td {
  padding: 10px 14px;
  border-bottom: 1px solid #eef3f9;
  vertical-align: middle;
  color: #16325c;
  font-size: 13px;
}

/* clickable row hover: change entire row background */
.cq-row {
  cursor: pointer;
  transition: background-color 0.15s ease;
}
.cq-row:hover td {
  background-color: #f3f8ff;
}

/* badge style for pending (yellow) */
.badge.bg-warning {
  background-color: #fff3cd !important;
  color: #856404 !important;
  padding: 6px 8px;
  border-radius: 6px;
  font-weight: 700;
  font-size: 12px;
}

/* compact pagination */
.pagination-sm .page-link {
  padding: 4px 8px;
  font-size: 13px;
}

/* make table responsive with rounded corners on small screens */
@media (max-width: 900px) {
  .cq-search-input {
    width: 100%;
    margin-top: 10px;
  }
}
```
