# PHTTv1
Transfer Tax Calculator Project
import React, { useState, useEffect } from 'react';

export default function TransferTaxCalculator() {
  const [taxBase, setTaxBase] = useState('');
  const [transferType, setTransferType] = useState('Sale');
  const [notaryDate, setNotaryDate] = useState('');
  const [deathDate, setDeathDate] = useState('');
  const [paymentDate, setPaymentDate] = useState('');
  const [results, setResults] = useState(null);

  const rates = [0.00500, 0.00550, 0.00600, 0.00650, 0.00750, 0.00825];

  const calculateMonthsLate = () => {
    if (!paymentDate) return 0;

    const payment = new Date(paymentDate);
    let startDate;

    if (transferType === 'EJS') {
      if (!deathDate) return 0;
      startDate = new Date(deathDate);
    } else {
      if (!notaryDate) return 0;
      startDate = new Date(notaryDate);
      startDate.setDate(startDate.getDate() + 60);
    }

    if (payment <= startDate) return 0;

    const diffTime = payment - startDate;
    const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    const months = Math.ceil(diffDays / 30);
    
    return months;
  };

  const calculateResults = () => {
    const base = parseFloat(taxBase);
    if (isNaN(base) || base <= 0) {
      setResults(null);
      return;
    }

    const monthsLate = calculateMonthsLate();
    const penaltyRate = Math.min(monthsLate * 0.02, 0.72);

    const calculations = rates.map(rate => {
      const basicTT = base * rate;
      const surcharge = basicTT * 0.25;
      const penalty = (basicTT + surcharge) * penaltyRate;
      const total = basicTT + surcharge + penalty;

      return {
        rate,
        basicTT,
        surcharge,
        penalty,
        total
      };
    });

    setResults({
      calculations,
      monthsLate,
      penaltyRate
    });
  };

  useEffect(() => {
    calculateResults();
  }, [taxBase, transferType, notaryDate, deathDate, paymentDate]);

  const formatCurrency = (value) => {
    return new Intl.NumberFormat('en-PH', {
      style: 'currency',
      currency: 'PHP',
      minimumFractionDigits: 2,
      maximumFractionDigits: 2
    }).format(value);
  };

  return (
    <div className="min-h-screen bg-black text-white p-6">
      <div className="max-w-7xl mx-auto">
        <h1 className="text-2xl font-bold mb-6">Transfer Tax Calculator with Penalties</h1>
        
        <div className="bg-gray-900 rounded-lg p-6 mb-6">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
            <div>
              <label className="block mb-2">Transfer Type</label>
              <select
                value={transferType}
                onChange={(e) => setTransferType(e.target.value)}
                className="w-full bg-gray-800 text-white border border-gray-700 rounded px-3 py-2"
              >
                <option value="Sale">Sale</option>
                <option value="Donation">Donation</option>
                <option value="EJS">EJS (Extrajudicial Settlement)</option>
              </select>
            </div>

            <div>
              <label className="block mb-2">Tax Base (ZV/SP/MV)</label>
              <input
                type="number"
                step="0.01"
                value={taxBase}
                onChange={(e) => setTaxBase(e.target.value)}
                placeholder="0.00"
                className="w-full bg-gray-800 text-white border border-gray-700 rounded px-3 py-2"
              />
            </div>

            {transferType !== 'EJS' && (
              <div>
                <label className="block mb-2">Notary Date</label>
                <input
                  type="text"
                  value={notaryDate}
                  onChange={(e) => setNotaryDate(e.target.value)}
                  placeholder="MM/DD/YYYY"
                  className="w-full bg-gray-800 text-white border border-gray-700 rounded px-3 py-2"
                />
              </div>
            )}

            {transferType === 'EJS' && (
              <div>
                <label className="block mb-2">Death Date</label>
                <input
                  type="text"
                  value={deathDate}
                  onChange={(e) => setDeathDate(e.target.value)}
                  placeholder="MM/DD/YYYY"
                  className="w-full bg-gray-800 text-white border border-gray-700 rounded px-3 py-2"
                />
              </div>
            )}

            <div>
              <label className="block mb-2">Payment Date</label>
              <input
                type="text"
                value={paymentDate}
                onChange={(e) => setPaymentDate(e.target.value)}
                placeholder="MM/DD/YYYY"
                className="w-full bg-gray-800 text-white border border-gray-700 rounded px-3 py-2"
              />
            </div>
          </div>

          {results && (
            <div className="mt-4 p-3 bg-gray-800 rounded">
              <p>Months Late: <span className="font-bold">{results.monthsLate}</span></p>
              <p>Penalty Rate: <span className="font-bold">{(results.penaltyRate * 100).toFixed(0)}%</span></p>
            </div>
          )}
        </div>

        {results && (
          <div className="bg-gray-900 rounded-lg p-6 overflow-x-auto">
            <h2 className="text-xl font-bold mb-4">Transfer Tax Computation</h2>
            <table className="w-full text-left border-collapse">
              <thead>
                <tr className="border-b border-gray-700">
                  <th className="p-3">Particulars</th>
                  {rates.map((rate, idx) => (
                    <th key={idx} className="p-3 text-right">{rate.toFixed(5)}</th>
                  ))}
                </tr>
              </thead>
              <tbody>
                <tr className="border-b border-gray-700">
                  <td className="p-3">Basic TT</td>
                  {results.calculations.map((calc, idx) => (
                    <td key={idx} className="p-3 text-right">{formatCurrency(calc.basicTT)}</td>
                  ))}
                </tr>
                <tr className="border-b border-gray-700">
                  <td className="p-3">Surcharge</td>
                  {results.calculations.map((calc, idx) => (
                    <td key={idx} className="p-3 text-right">{formatCurrency(calc.surcharge)}</td>
                  ))}
                </tr>
                <tr className="border-b border-gray-700">
                  <td className="p-3">Penalties</td>
                  {results.calculations.map((calc, idx) => (
                    <td key={idx} className="p-3 text-right">{formatCurrency(calc.penalty)}</td>
                  ))}
                </tr>
                <tr className="bg-green-900">
                  <td className="p-3 font-bold text-lg">Total</td>
                  {results.calculations.map((calc, idx) => (
                    <td key={idx} className="p-3 text-right font-bold text-lg text-green-400">
                      {formatCurrency(calc.total)}
                    </td>
                  ))}
                </tr>
              </tbody>
            </table>
          </div>
        )}
      </div>
    </div>
  );
}
