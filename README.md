# Color-Picker-Chrome-Extension
- Developed a Chrome extension that enables users to easily pick colors from any web page.
- Implemented the extension using HTML, CSS, and JavaScript, utilizing Chrome Extension APIs.
- Designed an intuitive user interface with a color picker tool and real-time color preview.
- Integrated functionality to copy the selected color code to the clipboard.
```
import React, { useState } from 'react';
import './CheckerQueue.css';

const CheckerQueue = () => {
  const [searchTerm, setSearchTerm] = useState('');

  // Dummy data with Timestamp datatype for dates
  const workItems = [
    {
      workflowId: 'WF001',
      loanId: 'LN2025001',
      userId: 'USR123',
      applicantName: 'John Doe',
      status: 'pending',
      created: new Date('2025-09-12T10:30:00Z')
    },
    {
      workflowId: 'WF002',
      loanId: 'LN2025002',
      userId: 'USR456',
      applicantName: 'Alice Smith',
      status: 'pending',
      created: new Date('2025-09-15T14:20:00Z')
    },
    {
      workflowId: 'WF003',
      loanId: 'LN2025003',
      userId: 'USR789',
      applicantName: 'Rahul Kumar',
      status: 'pending',
      created: new Date('2025-09-10T09:15:00Z')
    },
    {
      workflowId: 'WF004',
      loanId: 'LN2025004',
      userId: 'USR321',
      applicantName: 'Priya Verma',
      status: 'pending',
      created: new Date('2025-09-14T16:45:00Z')
    },
    {
      workflowId: 'WF005',
      loanId: 'LN2025005',
      userId: 'USR654',
      applicantName: 'Michael Johnson',
      status: 'pending',
      created: new Date('2025-09-13T11:30:00Z')
    }
  ];

  // Format date to "12 Sep 2025" format
  const formatDate = (timestamp) => {
    const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
                   'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
    const date = new Date(timestamp);
    const day = date.getDate();
    const month = months[date.getMonth()];
    const year = date.getFullYear();
    return `${day} ${month} ${year}`;
  };

  // Filter items based on search term
  const filteredItems = workItems.filter(item => {
    const searchLower = searchTerm.toLowerCase();
    const dateFormatted = formatDate(item.created).toLowerCase();
    
    return item.workflowId.toLowerCase().includes(searchLower) ||
           item.loanId.toLowerCase().includes(searchLower) ||
           dateFormatted.includes(searchLower);
  });

  const handleView = (workflowId) => {
    alert(`Viewing details for Workflow ID: ${workflowId}`);
    // Add your view logic here
  };

  return (
    <div className="checker-queue-container">
      <div className="container-fluid p-4">
        {/* Header */}
        <div className="mb-4">
          <h2 className="checker-queue-title">Checker Queue</h2>
        </div>

        {/* Search Bar */}
        <div className="row mb-4">
          <div className="col-md-6">
            <div className="search-container">
              <input
                type="text"
                className="form-control search-input"
                placeholder="Search by Workflow ID, Loan ID, or Date (e.g., 12 Sep 2025)"
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
              />
              <i className="fas fa-search search-icon"></i>
            </div>
          </div>
        </div>

        {/* Table */}
        <div className="table-container">
          <table className="table table-hover rounded-table">
            <thead className="table-header">
              <tr>
                <th scope="col">Workflow ID</th>
                <th scope="col">Loan ID</th>
                <th scope="col">User ID</th>
                <th scope="col">Applicant Name</th>
                <th scope="col">Status</th>
                <th scope="col">Created</th>
                <th scope="col">Actions</th>
              </tr>
            </thead>
            <tbody>
              {filteredItems.map((item, index) => (
                <tr key={item.workflowId} className="table-row">
                  <td className="fw-semibold">{item.workflowId}</td>
                  <td>{item.loanId}</td>
                  <td>{item.userId}</td>
                  <td>{item.applicantName}</td>
                  <td>
                    <span className="badge status-pending">
                      PENDING
                    </span>
                  </td>
                  <td className="text-muted">{formatDate(item.created)}</td>
                  <td>
                    <button
                      className="btn btn-view"
                      onClick={() => handleView(item.workflowId)}
                    >
                      View
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>

          {/* Empty State */}
          {filteredItems.length === 0 && (
            <div className="empty-state">
              <p className="text-muted">No items found matching your search criteria.</p>
            </div>
          )}
        </div>

        {/* Results Count */}
        <div className="mt-3">
          <small className="text-muted">
            Showing {filteredItems.length} of {workItems.length} items
          </small>
        </div>
      </div>

      <style jsx>{`
        .checker-queue-container {
          background-color: #f8f9fa;
          min-height: 100vh;
        }

        .checker-queue-title {
          color: #2c3e50;
          font-weight: 600;
          margin-bottom: 0;
        }

        .search-container {
          position: relative;
        }

        .search-input {
          border: 2px solid #e9ecef;
          border-radius: 8px;
          padding: 12px 45px 12px 15px;
          font-size: 14px;
          transition: all 0.3s ease;
        }

        .search-input:focus {
          border-color: #007bff;
          box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
        }

        .search-icon {
          position: absolute;
          right: 15px;
          top: 50%;
          transform: translateY(-50%);
          color: #6c757d;
        }

        .table-container {
          background: white;
          border-radius: 12px;
          overflow: hidden;
          box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
        }

        .rounded-table {
          margin-bottom: 0;
          border-radius: 12px;
          overflow: hidden;
        }

        .table-header {
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          color: white;
        }

        .table-header th {
          border: none;
          padding: 15px 20px;
          font-weight: 600;
          font-size: 14px;
          letter-spacing: 0.5px;
          text-transform: uppercase;
        }

        .table-row {
          transition: all 0.3s ease;
        }

        .table-row:hover {
          background-color: #f8f9ff;
          transform: translateY(-1px);
        }

        .table-row td {
          padding: 15px 20px;
          border-bottom: 1px solid #e9ecef;
          vertical-align: middle;
        }

        .status-pending {
          background: linear-gradient(135deg, #ffeaa7, #fab1a0);
          color: #d63031;
          font-weight: 600;
          font-size: 11px;
          letter-spacing: 0.5px;
          padding: 6px 12px;
          border-radius: 20px;
          border: none;
        }

        .btn-view {
          background: linear-gradient(135deg, #74b9ff, #0984e3);
          color: white;
          border: none;
          padding: 8px 20px;
          border-radius: 20px;
          font-size: 13px;
          font-weight: 500;
          transition: all 0.3s ease;
          box-shadow: 0 2px 5px rgba(116, 185, 255, 0.3);
        }

        .btn-view:hover {
          background: linear-gradient(135deg, #0984e3, #74b9ff);
          transform: translateY(-2px);
          box-shadow: 0 4px 15px rgba(116, 185, 255, 0.4);
          color: white;
        }

        .empty-state {
          text-align: center;
          padding: 60px 20px;
          background: white;
        }

        .fw-semibold {
          font-weight: 600;
          color: #2c3e50;
        }

        .text-muted {
          color: #6c757d !important;
        }

        /* Responsive Design */
        @media (max-width: 768px) {
          .table-container {
            overflow-x: auto;
          }
          
          .rounded-table {
            min-width: 800px;
          }
          
          .search-input {
            font-size: 16px; /* Prevents zoom on iOS */
          }
        }

        /* Custom scrollbar for table */
        .table-container::-webkit-scrollbar {
          height: 8px;
        }

        .table-container::-webkit-scrollbar-track {
          background: #f1f1f1;
          border-radius: 4px;
        }

        .table-container::-webkit-scrollbar-thumb {
          background: #c1c1c1;
          border-radius: 4px;
        }

        .table-container::-webkit-scrollbar-thumb:hover {
          background: #a8a8a8;
        }

        /* Pagination Styles */
        .pagination .page-link {
          color: #667eea;
          border: 1px solid #dee2e6;
          padding: 8px 12px;
          margin: 0 2px;
          border-radius: 6px;
          font-size: 14px;
          transition: all 0.3s ease;
        }

        .pagination .page-link:hover {
          color: #fff;
          background: linear-gradient(135deg, #667eea, #764ba2);
          border-color: #667eea;
          transform: translateY(-1px);
        }

        .pagination .page-item.active .page-link {
          background: linear-gradient(135deg, #667eea, #764ba2);
          border-color: #667eea;
          color: white;
          box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
        }

        .pagination .page-item.disabled .page-link {
          color: #6c757d;
          background-color: #fff;
          border-color: #dee2e6;
          cursor: not-allowed;
        }

        .pagination .page-item.disabled .page-link:hover {
          transform: none;
          background-color: #fff;
        }
      `}</style>

      {/* Bootstrap CSS CDN */}
      <link
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
        rel="stylesheet"
      />
      {/* Font Awesome for search icon */}
      <link
        rel="stylesheet"
        href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css"
      />
    </div>
  );
};

export default CheckerQueue;
```
