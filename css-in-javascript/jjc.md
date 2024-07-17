"use client";
import React from "react";
import * as XLSX from 'xlsx';

function MainComponent() {
  const [activeTab, setActiveTab] = React.useState("商品管理");
  const [exchangeRates, setExchangeRates] = React.useState({
    USDJPY: 0,
    EURJPY: 0,
    GBPJPY: 0,
  });
  const [products, setProducts] = React.useState([]);
  const [workflow, setWorkflow] = React.useState([]);
  const [schedule, setSchedule] = React.useState([]);
  const [contacts, setContacts] = React.useState({
    customers: [],
    suppliers: [],
    organizations: [],
  });
  const [financialData, setFinancialData] = React.useState({
    revenue: 0,
    expenses: 0,
    profit: 0,
    taxRate: 0.3,
    taxAmount: 0
  });
  const [isLoggedIn, setIsLoggedIn] = React.useState(false);
  const [selectedFile, setSelectedFile] = React.useState(null);

  React.useEffect(() => {
    const fetchExchangeRates = async () => {
      try {
        const response = await fetch("https://api.exchangerate-api.com/v4/latest/JPY");
        const data = await response.json();
        setExchangeRates({
          USDJPY: (1 / data.rates.USD).toFixed(2),
          EURJPY: (1 / data.rates.EUR).toFixed(2),
          GBPJPY: (1 / data.rates.GBP).toFixed(2),
        });
      } catch (error) {
        console.error("Error fetching exchange rates:", error);
      }
    };

    fetchExchangeRates();
    const interval = setInterval(fetchExchangeRates, 60000);

    return () => clearInterval(interval);
  }, []);

  const handleLogin = (event) => {
    event.preventDefault();
    const username = event.target.username.value;
    const password = event.target.password.value;
    if (username === "jjc" && password === "123") {
      setIsLoggedIn(true);
    } else {
      alert("Invalid credentials");
    }
  };

  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    setSelectedFile(file);
  };

  const handleFileAnalysis = () => {
    if (selectedFile) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        const json = XLSX.utils.sheet_to_json(worksheet);
        setProducts(json);
      };
      reader.readAsArrayBuffer(selectedFile);
    }
  };

  const handleWorkflowSubmit = (event) => {
    event.preventDefault();
    const newWorkflow = {
      id: Date.now(),
      name: event.target.workflowName.value,
      steps: event.target.workflowSteps.value.split(",").map((step) => step.trim()),
    };
    setWorkflow([...workflow, newWorkflow]);
    event.target.reset();
  };

  const handleScheduleSubmit = (event) => {
    event.preventDefault();
    const newSchedule = {
      id: Date.now(),
      date: event.target.date.value,
      event: event.target.event.value,
      location: event.target.location.value,
      budget: parseFloat(event.target.budget.value),
      task: event.target.task.value,
    };
    setSchedule([...schedule, newSchedule]);
    event.target.reset();
  };

  const handleContactSubmit = (event, type) => {
    event.preventDefault();
    const newContact = {
      id: Date.now(),
      name: event.target.name.value,
      phone: event.target.phone.value,
      email: event.target.email.value,
      contact: event.target.contact.value,
      website: event.target.website.value,
    };
    setContacts((prev) => ({
      ...prev,
      [type]: [...prev[type], newContact],
    }));
    event.target.reset();
  };

  const handleFinancialUpdate = (event) => {
    event.preventDefault();
    const revenue = parseFloat(event.target.revenue.value);
    const expenses = parseFloat(event.target.expenses.value);
    const profit = revenue - expenses;
    const taxAmount = profit * financialData.taxRate;
    setFinancialData({
      ...financialData,
      revenue,
      expenses,
      profit,
      taxAmount,
    });
  };

  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-gray-100 flex items-center justify-center">
        <form onSubmit={handleLogin} className="bg-white p-8 rounded shadow-md">
          <h2 className="text-2xl font-bold mb-4">Login</h2>
          <input
            type="text"
            name="username"
            placeholder="Username"
            className="w-full p-2 mb-4 border rounded"
            required
          />
          <input
            type="password"
            name="password"
            placeholder="Password"
            className="w-full p-2 mb-4 border rounded"
            required
          />
          <button
            type="submit"
            className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
          >
            Login
          </button>
        </form>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 font-sans">
      <header className="bg-blue-600 text-white p-4 flex justify-between items-center">
        <h1 className="text-2xl font-bold">公司内部交互網站</h1>
        <div className="flex space-x-4 text-sm">
          <div>USD/JPY: {exchangeRates.USDJPY}</div>
          <div>EUR/JPY: {exchangeRates.EURJPY}</div>
          <div>GBP/JPY: {exchangeRates.GBPJPY}</div>
        </div>
      </header>

      <nav className="bg-blue-500 p-4">
        <ul className="flex flex-wrap space-x-4">
          {["商品管理", "工作流程", "計劃表", "顧客管理", "供應商管理", "外部機構", "財務統計"].map((tab) => (
            <li key={tab}>
              <a
                href="#"
                className={`text-white hover:text-blue-200 ${activeTab === tab ? "font-bold" : ""}`}
                onClick={() => setActiveTab(tab)}
              >
                {tab}
              </a>
            </li>
          ))}
        </ul>
      </nav>

      <main className="p-8">
        {activeTab === "商品管理" && (
          <section className="mb-8">
            <h2 className="text-xl font-semibold mb-4">商品管理</h2>
            <div className="bg-white p-4 rounded shadow">
              <input
                type="file"
                accept=".xlsx, .xls, .csv, .txt"
                className="mb-4"
                onChange={handleFileUpload}
              />
              <button
                className="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
                onClick={handleFileAnalysis}
              >
                分析商品信息
              </button>
              {products.length > 0 && (
                <div className="mt-4 overflow-x-auto">
                  <table className="min-w-full bg-white">
                    <thead>
                      <tr>
                        <th className="px-4 py-2 border">中文名</th>
                        <th className="px-4 py-2 border">保質期（日）</th>
                        <th className="px-4 py-2 border">地域</th>
                        <th className="px-4 py-2 border">試食郵寄</th>
                        <th className="px-4 py-2 border">最低入貨數（個/組）</th>
                        <th className="px-4 py-2 border">出口價（英鎊）</th>
                      </tr>
                    </thead>
                    <tbody>
                      {products.map((product, index) => (
                        <tr key={index}>
                          <td className="px-4 py-2 border">{product["中文名"]}</td>
                          <td className="px-4 py-2 border">{product["保質期（日）"]}</td>
                          <td className="px-4 py-2 border">{product["地域"]}</td>
                          <td className="px-4 py-2 border">{product["試食郵寄"]}</td>
                          <td className="px-4 py-2 border">{product["最低入貨數（個/組）"]}</td>
                          <td className="px-4 py-2 border">{product["出口價（英鎊）"]}</td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              )}
            </div>
          </section>
        )}

        {activeTab === "工作流程" && (
          <section className="mb-8">
            <h2 className="text-xl font-semibold mb-4">自定義工作流程</h2>
            <div className="bg-white p-4 rounded shadow">
              <form onSubmit={handleWorkflowSubmit} className="space-y-4">
                <input
                  type="text"
                  name="workflowName"
                  className="w-full p-2 border rounded"
                  placeholder="工作流程名稱"
                  required
                />
                <textarea
                  name="workflowSteps"
                  className="w-full p-2 border rounded"
                  placeholder="步驟（用逗號分隔）"
                  rows="3"
                  required
                ></textarea>
                <button
                  type="submit"
                  className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
                >
                  添加工作流程
                </button>
              </form>
              {workflow.length > 0 && (
                <div className="mt-4">
                  <h3 className="font-semibold mb-2">已定義的工作流程：</h3>
                  <ul className="list-disc pl-5">
                    {workflow.map((flow) => (
                      <li key={flow.id}>
                        {flow.name}: {flow.steps.join(" → ")}
                      </li>
                    ))}
                  </ul>
                </div>
              )}
            </div>
          </section>
        )}

        {activeTab === "計劃表" && (
          <section className="mb-8">
            <h2 className="text-xl font-semibold mb-4">公司計劃表</h2>
            <div className="bg-white p-4 rounded shadow">
              <form onSubmit={handleScheduleSubmit} className="space-y-4">
                <input
                  type="date"
                  name="date"
                  className="w-full p-2 border rounded"
                  placeholder="時間"
                  required
                />
                <input
                  type="text"
                  name="event"
                  className="w-full p-2 border rounded"
                  placeholder="事情"
                  required
                />
                <input
                  type="text"
                  name="location"
                  className="w-full p-2 border rounded"
                  placeholder="地點"
                  required
                />
                <input
                  type="number"
                  name="budget"
                  className="w-full p-2 border rounded"
                  placeholder="預算"
                  required
                />
                <textarea
                  name="task"
                  className="w-full p-2 border rounded"
                  placeholder="任務"
                  rows="3"
                  required
                ></textarea>
                <button
                  type="submit"
                  className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
                >
                  添加計劃
                </button>
              </form>
              {schedule.length > 0 && (
                <div className="mt-4">
                  <h3 className="font-semibold mb-2">已添加的計劃：</h3>
                  <ul className="list-disc pl-5">
                    {schedule.map((item) => (
                      <li key={item.id}>
                        {item.date}: {item.event} at {item.location} (¥{item.budget})
                      </li>
                    ))}
                  </ul>
                </div>
              )}
            </div>
          </section>
        )}

        {(activeTab === "顧客管理" ||
          activeTab === "供應商管理" ||
          activeTab === "外部機構") && (
          <section className="mb-8">
            <h2 className="text-xl font-semibold mb-4">聯繫人管理</h2>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div className="bg-white p-4 rounded shadow">
                <h3 className="font-semibold mb-2">{activeTab}</h3>
                <form
                  onSubmit={(e) =>
                    handleContactSubmit(
                      e,
                      activeTab === "顧客管理"
                        ? "customers"
                        : activeTab === "供應商管理"
                        ? "suppliers"
                        : "organizations"
                    )
                  }
                  className="space-y-2"
                >
                  <input
                    type="text"
                    name="name"
                    className="w-full p-2 border rounded"
                    placeholder="名稱"
                    required
                  />
                  <input
                    type="tel"
                    name="phone"
                    className="w-full p-2 border rounded"
                    placeholder="電話"
                    required
                  />
                  <input
                    type="email"
                    name="email"
                    className="w-full p-2 border rounded"
                    placeholder="郵件"
                    required
                  />
                  <input
                    type="text"
                    name="contact"
                    className="w-full p-2 border rounded"
                    placeholder="對接人物"
                    required
                  />
                  <input
                    type="url"
                    name="website"
                    className="w-full p-2 border rounded"
                    placeholder="網站"
                    required
                  />
                  <button
                    type="submit"
                    className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
                  >
                    添加
                  </button>
                </form>
              </div>
              <div className="bg-white p-4 rounded shadow col-span-2">
                <h3 className="font-semibold mb-2">聯繫人列表</h3>
                <ul className="list-disc pl-5">
                  {contacts[
                    activeTab === "顧客管理"
                      ? "customers"
                      : activeTab === "供應商管理"
                      ? "suppliers"
                      : "organizations"
                  ].map((contact) => (
                    <li key={contact.id}>
                      {contact.name} - {contact.phone} - {contact.email}
                    </li>
                  ))}
                </ul>
              </div>
            </div>
          </section>
        )}

        {activeTab === "財務統計" && (
          <section className="mb-8">
            <h2 className="text-xl font-semibold mb-4">財務統計</h2>
            <div className="bg-white p-4 rounded shadow">
              <form onSubmit={handleFinancialUpdate} className="space-y-4">
                <input
                  type="number"
                  name="revenue"
                  className="w-full p-2 border rounded"
                  placeholder="收入"
                  required
                />
                <input
                  type="number"
                  name="expenses"
                  className="w-full p-2 border rounded"
                  placeholder="支出"
                  required
                />
                <button
                  type="submit"
                  className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
                >
                  更新財務數據
                </button>
              </form>
              <div className="mt-4">
                <h3 className="font-semibold mb-2">當前財務數據：</h3>
                <p>收入: ¥{financialData.revenue}</p>
                <p>支出: ¥{financialData.expenses}</p>
                <p>利潤: ¥{financialData.profit}</p>
                <p>稅款: ¥{financialData.taxAmount}</p>
              </div>
            </div>
          </section>
        )}
      </main>
    </div>
  );
}

export default MainComponent;
