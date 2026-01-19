# Spinner / Loading States

ASP.NET Zero React UI uses [Ant Design's Spin component](https://ant.design/components/spin) for showing loading states.

## Route Loading Spinner

When lazy-loaded routes are being loaded, a centered spinner is shown. This is implemented in [AppRouter.tsx](../src/routes/AppRouter.tsx):

```tsx
import { Spin } from "antd";

const LoadingSpinner = () => (
  <div
    style={{
      display: "flex",
      justifyContent: "center",
      alignItems: "center",
      height: "100vh",
    }}
  >
    <Spin size="large" />
  </div>
);

// Used with React.Suspense
<Suspense fallback={<LoadingSpinner />}>
  <Routes>
    {/* ... */}
  </Routes>
</Suspense>
```

## Table Loading States

Ant Design's Table component has a built-in `loading` prop:

```tsx
import { Table } from "antd";

const MyTable: React.FC = () => {
  const [loading, setLoading] = useState(false);
  const [data, setData] = useState([]);

  const fetchData = async () => {
    setLoading(true);
    try {
      const result = await service.getData();
      setData(result);
    } finally {
      setLoading(false);
    }
  };

  return <Table dataSource={data} loading={loading} columns={columns} />;
};
```

## Element-Level Spinners

For wrapping specific elements with a loading overlay:

```tsx
import { Spin } from "antd";

const MyComponent: React.FC = () => {
  const [loading, setLoading] = useState(true);

  return (
    <Spin spinning={loading}>
      <div className="card">
        {/* Content that will be covered by spinner */}
      </div>
    </Spin>
  );
};
```

## Customizing Spinners

You can customize the Spin component appearance using Ant Design's props:

```tsx
<Spin 
  size="large"  // "small" | "default" | "large"
  tip="Loading..." // Optional loading text
/>
```

Check [Ant Design Spin documentation](https://ant.design/components/spin) for more customization options.
