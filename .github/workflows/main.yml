import React, { useState } from "react";
import { saveAs } from "file-saver";
import * as XLSX from "xlsx";

// Default export React component
export default function VehicleLoadingApp() {
  const [rows, setRows] = useState([]);
  const [template, setTemplate] = useState(
    "Aapki vehicle load ho rahi hai. Driver: {Driver Name} ({Driver Mobile}), Vehicle: {Vehicle Number} - Date: {Loading Date} {Loading Time}."
  );

  // Excel/CSV file handler (uses SheetJS)
  const handleFile = async (e) => {
    const file = e.target.files[0];
    if (!file) return;
    const data = await file.arrayBuffer();
    const workbook = XLSX.read(data);
    const sheetName = workbook.SheetNames[0];
    const worksheet = workbook.Sheets[sheetName];
    const json = XLSX.utils.sheet_to_json(worksheet, { defval: "" });

    // Normalize keys (trim) and keep important columns
    const normalized = json.map((r, i) => ({
      id: r['Dealer ID'] || r['DealerId'] || r['Dealer'] || `DL${i + 1}`,
      dealerName: r['Dealer Name'] || r['DealerName'] || r['Dealer'] || "",
      dealerMobile: String(r['Dealer Mobile Number'] || r['Dealer Mobile'] || r['Dealer Number'] || r['DealerPhone'] || "").replace(/\s+/g, ""),
      district: r['District'] || "",
      vehicleNumber: r['Vehicle Number'] || r['VehicleNo'] || r['Vehicle'] || "",
      driverName: r['Driver Name'] || r['Driver'] || "",
      driverMobile: String(r['Driver Mobile Number'] || r['Driver Mobile'] || r['DriverPhone'] || "").replace(/\s+/g, ""),
      loadingDate: r['Loading Date'] || r['Date'] || "",
      loadingTime: r['Loading Time'] || r['Time'] || "",
    }));

    // Pre-generate messages
    const withMsg = normalized.map((r) => ({ ...r, message: fillTemplate(template, r) }));
    setRows(withMsg);
  };

  // Fill template with row values
  function fillTemplate(tpl, row) {
    return tpl
      .replaceAll('{Dealer Name}', row.dealerName || '')
      .replaceAll('{Dealer Mobile}', row.dealerMobile || '')
      .replaceAll('{Vehicle Number}', row.vehicleNumber || '')
      .replaceAll('{Driver Name}', row.driverName || '')
      .replaceAll('{Driver Mobile}', row.driverMobile || '')
      .replaceAll('{Loading Date}', row.loadingDate || '')
      .replaceAll('{Loading Time}', row.loadingTime || '');
  }

  // When template changes, update messages
  const handleTemplateChange = (e) => {
    const t = e.target.value;
    setTemplate(t);
    setRows((prev) => prev.map((r) => ({ ...r, message: fillTemplate(t, r) })));
  };

  // Open WhatsApp web with prefilled message (opens in new tab)
  const sendWhatsApp = (phone, text) => {
    if (!phone) {
      alert('Dealer ka mobile number missing hai');
      return;
    }
    // Ensure phone is in international format if possible. If user has local number (10 digits), we'll prefix +91 by default.
    let p = phone.replace(/[^0-9+]/g, '');
    if (!p.startsWith('+')) {
      // assume India if 10 digits
      if (p.length === 10) p = '+91' + p;
    }
    const url = `https://api.whatsapp.com/send?phone=${encodeURIComponent(p)}&text=${encodeURIComponent(text)}`;
    window.open(url, '_blank');
  };

  // Copy message to clipboard
  const copyMessage = async (text) => {
    try {
      await navigator.clipboard.writeText(text);
      alert('Message copied to clipboard');
    } catch (e) {
      alert('Copy failed');
    }
  };

  // Export current rows to Excel
  const exportToExcel = () => {
    const ws = XLSX.utils.json_to_sheet(rows.map(r => ({
      'Dealer ID': r.id,
      'Dealer Name': r.dealerName,
      'Dealer Mobile Number': r.dealerMobile,
      'District': r.district,
      'Vehicle Number': r.vehicleNumber,
      'Driver Name': r.driverName,
      'Driver Mobile Number': r.driverMobile,
      'Loading Date': r.loadingDate,
      'Loading Time': r.loadingTime,
      'Message (Auto)': r.message,
    })));
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, 'Vehicle_Loading');
    const wbout = XLSX.write(wb, { bookType: 'xlsx', type: 'array' });
    const blob = new Blob([wbout], { type: 'application/octet-stream' });
    saveAs(blob, 'Vehicle_Loading_Export.xlsx');
  };

  return (
    <div className="min-h-screen bg-gray-50 p-6">
      <div className="max-w-5xl mx-auto bg-white shadow rounded-lg p-6">
        <h1 className="text-2xl font-semibold mb-4">Vehicle Loading - Send Messages to Dealers</h1>

        <p className="text-sm mb-4">Excel file upload karein (first sheet) jisme Dealer aur Vehicle details hon. App message template ko customize kar sakte hain. Use placeholders: <code>{'{Driver Name}'}</code>, <code>{'{Driver Mobile}'}</code>, <code>{'{Vehicle Number}'}</code>, <code>{'{Dealer Name}'}</code>, <code>{'{Dealer Mobile}'}</code>, <code>{'{Loading Date}'}</code>, <code>{'{Loading Time}'}</code>.</p>

        <div className="mb-4 flex gap-4 items-center">
          <input type="file" accept=".xlsx,.xls,.csv" onChange={handleFile} className="p-2 border rounded" />
          <button onClick={exportToExcel} className="px-4 py-2 bg-blue-600 text-white rounded">Export Excel</button>
        </div>

        <label className="block text-sm font-medium mb-2">Message Template</label>
        <textarea value={template} onChange={handleTemplateChange} rows={3} className="w-full p-3 border rounded mb-4" />

        <div className="overflow-x-auto">
          <table className="w-full table-auto border-collapse">
            <thead>
              <tr className="bg-gray-100 text-left">
                <th className="p-2 border">#</th>
                <th className="p-2 border">Dealer</th>
                <th className="p-2 border">Dealer Mobile</th>
                <th className="p-2 border">Vehicle</th>
                <th className="p-2 border">Driver</th>
                <th className="p-2 border">Date / Time</th>
                <th className="p-2 border">Message</th>
                <th className="p-2 border">Actions</th>
              </tr>
            </thead>
            <tbody>
              {rows.length === 0 && (
                <tr><td colSpan={8} className="p-4 text-center text-gray-500">Koi data nahi mila. Excel upload karein ya sample add karein.</td></tr>
              )}
              {rows.map((r, idx) => (
                <tr key={idx} className="odd:bg-white even:bg-gray-50">
                  <td className="p-2 border align-top">{idx + 1}</td>
                  <td className="p-2 border align-top">{r.dealerName}</td>
                  <td className="p-2 border align-top">{r.dealerMobile}</td>
                  <td className="p-2 border align-top">{r.vehicleNumber}</td>
                  <td className="p-2 border align-top">{r.driverName} <br /> {r.driverMobile}</td>
                  <td className="p-2 border align-top">{r.loadingDate} {r.loadingTime}</td>
                  <td className="p-2 border align-top"><div className="whitespace-pre-wrap">{r.message}</div></td>
                  <td className="p-2 border align-top flex gap-2 flex-col">
                    <button onClick={() => sendWhatsApp(r.dealerMobile, r.message)} className="px-2 py-1 rounded bg-green-600 text-white">WhatsApp</button>
                    <button onClick={() => copyMessage(r.message)} className="px-2 py-1 rounded bg-gray-700 text-white">Copy</button>
                    <a className="px-2 py-1 rounded bg-indigo-600 text-white text-center" href={`sms:${r.dealerMobile}?body=${encodeURIComponent(r.message)}`}>Send SMS</a>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>

        <div className="mt-6 text-sm text-gray-600">
          <p><strong>Note:</strong> WhatsApp link opens WhatsApp Web / App. SMS link works on phones. Website is frontend-only — agar aap chahte hain ki messages automatically send ho bina user interaction ke, to hume backend SMS/WhatsApp provider (Twilio, Gupshup, etc.) ki zarurat padegi aur uske liye API keys chahiye honge.</p>
        </div>
      </div>
    </div>
  );
}
