# Color-Picker-Chrome-Extension
- Developed a Chrome extension that enables users to easily pick colors from any web page.
- Implemented the extension using HTML, CSS, and JavaScript, utilizing Chrome Extension APIs.
- Designed an intuitive user interface with a color picker tool and real-time color preview.
- Integrated functionality to copy the selected color code to the clipboard.
```
import React, { useEffect, useMemo, useState } from "react";
import "./CheckerQueueComponent.css"; // keep your existing CSS file

const PAGE_SIZE = 10;

// Helper: format ISO timestamp/string -> "12 Sep, 2025"
function formatDateIsoOnly(iso) {
  if (!iso) return "";
  const d = new Date(iso);
  const opts = { day: "2-digit", month: "short", year: "numeric" };
  // e.g. "12 Sep 2025"
  const parts = new Intl.DateTimeFormat("en-GB", opts).format(d);
  const [day, month, year] = parts.split(" ");
  return `${day} ${month}, ${year}`;
}

export default function CheckerQueueComponent() {
  const [rows, setRows] = useState([]);        // merged rows used by table
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // UI states (keep same search + pagination behavior)
  const [query, setQuery] = useState("");
  const [page, setPage] = useState(1);

  useEffect(() => {
    let cancelled = false;

    async function loadData() {
      setLoading(true);
      setError(null);

      try {
        // 1) Fetch workflows moved to checker
        // CHANGE this URL to your real workflows endpoint if needed
        const wfRes = await fetch("http://localhost:8080/getWorkflowsByStatus/Moved_To_Checker", {
          headers: { Accept: "application/json" },
        });
        if (!wfRes.ok) throw new Error(`Workflows fetch failed: ${wfRes.status}`);
        const workflows = await wfRes.json(); // array of workflow objects

        // 2) Extract unique user_ids
        const userIds = Array.from(new Set(workflows.map((w) => w.user_id).filter(Boolean)));

        // 3) For each userId call the API that returns a plain string (applicant name).
        // We fetch all in parallel with Promise.all
        const userPromises = userIds.map(async (uid) => {
          try {
            const r = await fetch(`http://localhost:8080/getCustomerByUserId/${encodeURIComponent(uid)}`, {
              headers: { Accept: "text/plain, application/json" },
            });
            if (!r.ok) {
              return { user_id: uid, name: "" };
            }
            // Endpoint returns plain string -> use text()
            const text = await r.text();
            return { user_id: uid, name: (text || "").trim() };
          } catch (err) {
            return { user_id: uid, name: "" };
          }
        });

        const userResults = await Promise.all(userPromises);
        const userMap = userResults.reduce((m, u) => {
          m[u.user_id] = u.name || "";
          return m;
        }, {});

        // 4) Build the final rows array used by the table.
        // Keep id formatting like WI-xxxx and LN-xxxx (this matches the UI you used)
        const merged = workflows.map((w, idx) => ({
          workitemId:
            w.workFlowId != null ? `WI-${1000 + Number(w.workFlowId)}` : `WI-${1000 + idx}`,
          loanId: w.loan_id != null ? `LN-${1000 + Number(w.loan_id)}` : `LN-${1000 + idx}`,
          userId: w.user_id ?? "",
          applicant: userMap[w.user_id] || "",
          status: w.status ?? "PENDING",
          createdAt: w.createdAt ?? w.created_at ?? w.CreatedAt ?? "",
          _raw: w,
        }));

        if (!cancelled) {
          setRows(merged);
          setLoading(false);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message || "Failed to load data");
          setLoading(false);
        }
      }
    }

    loadData();

    return () => {
      cancelled = true;
    };
  }, []); // run once on mount

  // --------------------------
  // Search (unchanged behavior): search across workitemId, loanId, userId, applicant, created date
  const filtered = useMemo(() => {
    const q = (query || "").trim().toLowerCase();
    if (!q) return rows;
    return rows.filter((r) => {
      if ((r.workitemId || "").toLowerCase().includes(q)) return true;
      if ((r.loanId || "").toLowerCase().includes(q)) return true;
      if ((r.userId || "").toLowerCase().includes(q)) return true;
      if ((r.applicant || "").toLowerCase().includes(q)) return true;
      if ((formatDateIsoOnly(r.createdAt) || "").toLowerCase().includes(q)) return true;
      return false;
    });
  }, [rows, query]);

  // --------------------------
  // Pagination (10 per page)
  const totalPages = Math.max(1, Math.ceil(filtered.length / PAGE_SIZE));
  const currentPage = Math.min(Math.max(1, page), totalPages);
  const pageStart = (currentPage - 1) * PAGE_SIZE;
  const pageRows = filtered.slice(pageStart, pageStart + PAGE_SIZE);

  // When search changes, go back to page 1
  useEffect(() => {
    setPage(1);
  }, [query]);

  // Simple view handler (replace with modal/navigation if you want)
  const handleView = (row) => {
    // keep simple: log to console (UI unchanged)
    console.log("VIEW clicked", row.workitemId, row);
    alert(`View ${row.workitemId}`);
  };

  // --------------------------
  // Render
  if (loading) return <div className="text-center p-4">Loading...</div>;
  if (error) return <div className="text-center text-danger p-4">Error: {error}</div>;

  return (
    <div>
      {/* Keep your existing heading/search markup & classes unchanged in your app.
          Below is the table and pagination rendered with the same class names you use */}
      <div className="table-responsive">
        <table className="table cq-table mb-0">
          <thead>
            <tr>
              <th>WORKITEM ID</th>
              <th>LOAN ID</th>
              <th>USER ID</th>
              <th>APPLICANT NAME</th>
              <th>STATUS</th>
              <th>CREATED</th>
              <th>ACTION</th>
            </tr>
          </thead>

          <tbody>
            {pageRows.length === 0 ? (
              <tr>
                <td colSpan="7" className="text-center py-4 text-muted">
                  No records found
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
                  <td>{formatDateIsoOnly(r.createdAt)}</td>
                  <td>
                    <button className="btn-view" onClick={() => handleView(r)}>
                      VIEW
                    </button>
                  </td>
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>

      {/* Pagination controls (same style as before) */}
      <div className="d-flex justify-content-end align-items-center mt-3 gap-2">
        <div className="text-muted me-3">Showing {pageRows.length} of {filtered.length}</div>
        <nav aria-label="Page navigation">
          <ul className="pagination pagination-sm mb-0">
            <li className={`page-item ${currentPage === 1 ? "disabled" : ""}`}>
              <button className="page-link" onClick={() => setPage((p) => Math.max(1, p - 1))}>
                Previous
              </button>
            </li>
            <li className="page-item disabled">
              <span className="page-link">
                {currentPage} / {totalPages}
              </span>
            </li>
            <li className={`page-item ${currentPage === totalPages ? "disabled" : ""}`}>
              <button className="page-link" onClick={() => setPage((p) => Math.min(totalPages, p + 1))}>
                Next
              </button>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  );
}
```
