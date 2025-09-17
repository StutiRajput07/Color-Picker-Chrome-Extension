# Color-Picker-Chrome-Extension
- Developed a Chrome extension that enables users to easily pick colors from any web page.
- Implemented the extension using HTML, CSS, and JavaScript, utilizing Chrome Extension APIs.
- Designed an intuitive user interface with a color picker tool and real-time color preview.
- Integrated functionality to copy the selected color code to the clipboard.
```
import React, { useEffect, useState } from "react";
import "./CheckerQueueComponent.css";

export default function CheckerQueueComponent() {
  const [rows, setRows] = useState([]);
  const [search, setSearch] = useState("");
  const [currentPage, setCurrentPage] = useState(1);
  const rowsPerPage = 10;

  // 🔹 Format ISO date to "12 Sept 2025"
  const formatDateIsoOnly = (isoString) => {
    if (!isoString) return "";
    const date = new Date(isoString);
    return date.toLocaleDateString("en-GB", {
      day: "2-digit",
      month: "short",
      year: "numeric",
    });
  };

  // 🔹 Fetch workflows + applicant names
  useEffect(() => {
    async function fetchData() {
      try {
        const res = await fetch(
          "http://localhost:8080/getWorkFlowsByStatus/Moved_To_Checker"
        );
        const workflows = await res.json();

        // fetch applicant name for each workflow
        const withApplicants = await Promise.all(
          workflows.map(async (wf) => {
            try {
              const res2 = await fetch(
                `http://localhost:8080/getApplicantName/${wf.user_id}`
              );
              const applicant = await res2.text(); // API returns string
              return { ...wf, applicant };
            } catch {
              return { ...wf, applicant: "N/A" };
            }
          })
        );

        setRows(withApplicants);
      } catch (err) {
        console.error("Error fetching workflows", err);
      }
    }

    fetchData();
  }, []);

  // 🔹 Search filter (workitemId, loan_id, user_id, applicant, createdAt)
  const filteredRows = rows.filter((r) =>
    [r.workflowId, r.loan_id, r.user_id, r.applicant, formatDateIsoOnly(r.createdAt)]
      .join(" ")
      .toLowerCase()
      .includes(search.toLowerCase())
  );

  // 🔹 Pagination
  const totalPages = Math.ceil(filteredRows.length / rowsPerPage);
  const pageRows = filteredRows.slice(
    (currentPage - 1) * rowsPerPage,
    currentPage * rowsPerPage
  );

  // 🔹 Handle View click
  const handleView = (row) => {
    alert(`Viewing Workflow ID: ${row.workflowId}`);
  };

  return (
    <div className="cq-container">
      {/* Heading */}
      <div className="cq-header">
        <h2 className="cq-heading">CHECKER QUEUE</h2>
      </div>

      {/* Search */}
      <input
        type="text"
        className="form-control cq-search-input"
        placeholder="Search workitem / loan / user / applicant / date"
        value={search}
        onChange={(e) => {
          setSearch(e.target.value);
          setCurrentPage(1);
        }}
      />

      {/* Table */}
      <table className="table cq-table mt-3">
        <thead>
          <tr>
            <th>WORKFLOW ID</th>
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
              <tr key={r.workflowId} className="cq-row">
                <td className="fw-bold">{r.workflowId}</td>
                <td className="fw-bold">{r.loan_id}</td>
                <td>{r.user_id}</td>
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

      {/* Pagination */}
      <div className="cq-pagination">
        <button
          className="btn btn-sm btn-light"
          onClick={() => setCurrentPage((p) => Math.max(p - 1, 1))}
          disabled={currentPage === 1}
        >
          ◀
        </button>
        <span>
          Page {currentPage} of {totalPages || 1}
        </span>
        <button
          className="btn btn-sm btn-light"
          onClick={() => setCurrentPage((p) => Math.min(p + 1, totalPages))}
          disabled={currentPage === totalPages || totalPages === 0}
        >
          ▶
        </button>
      </div>
    </div>
  );
}

```
