# Creating Modal for New Person

We will create an Ant Design Modal to create a new person. ASP.NET Zero React version uses [Ant Design](https://ant.design/) library to create modals. Final modal will be like below:

![Create Person Dialog](images/phonebook-create-person-dialog-2.png)

First of all, we should use **nswag/refresh.bat** to re-generate service-proxies. This will generate the code that is needed to call PersonAppService.**CreatePerson** method from client side. Notice that you should rebuild & run the server side application before re-generating the proxy scripts.

We are starting from creating a new component, named **CreatePersonModal.tsx** into client side phonebook folder:

```typescript
import React, { useState, useEffect } from "react";
import { Modal } from "antd";
import { useForm } from "react-hook-form";
import {
  PersonServiceProxy,
  CreatePersonInput,
} from "@/api/service-proxy-factory";
import { useServiceProxy } from "@/api/service-proxy-factory";
import L from "@/lib/L";

interface Props {
  isVisible: boolean;
  onClose: () => void;
  onSave: () => void;
}

type FormValues = {
  name: string;
  surname: string;
  emailAddress?: string;
};

const CreatePersonModal: React.FC<Props> = ({ isVisible, onClose, onSave }) => {
  const {
    register,
    handleSubmit,
    reset,
    setFocus,
    formState: { errors, isValid },
  } = useForm<FormValues>({ mode: "onChange" });
  const [saving, setSaving] = useState(false);
  const personService = useServiceProxy(PersonServiceProxy, []);

  useEffect(() => {
    if (isVisible) {
      reset({ name: "", surname: "", emailAddress: "" });
    }
  }, [isVisible, reset]);

  const onSubmit = async (values: FormValues) => {
    setSaving(true);
    try {
      const input = new CreatePersonInput({
        name: values.name,
        surname: values.surname,
        emailAddress: values.emailAddress,
      });
      await personService.createPerson(input);
      abp.notify.success(L("SavedSuccessfully"));
      onSave();
      onClose();
    } finally {
      setSaving(false);
    }
  };

  return (
    <Modal
      title={L("CreateNewPerson")}
      open={isVisible}
      onCancel={onClose}
      afterOpenChange={(opened) => {
        if (opened) {
          setTimeout(() => setFocus("name"), 0);
        }
      }}
      footer={[
        <button
          key="cancel"
          type="button"
          className="btn btn-secondary"
          onClick={onClose}
          disabled={saving}
        >
          {L("Cancel")}
        </button>,
        <button
          key="save"
          type="submit"
          className="btn btn-primary"
          onClick={handleSubmit(onSubmit)}
          disabled={!isValid || saving}
        >
          {saving ? (
            <>
              <span
                className="spinner-border spinner-border-sm me-2"
                role="status"
                aria-hidden="true"
              ></span>
              {L("SavingWithThreeDot")}
            </>
          ) : (
            <>
              <i className="fa fa-save"></i> <span>{L("Save")}</span>
            </>
          )}
        </button>,
      ]}
    >
      <form onSubmit={handleSubmit(onSubmit)} noValidate>
        <div className="mb-5">
          <label className="form-label required" htmlFor="name">
            {L("Name")}
          </label>
          <input
            id="name"
            type="text"
            className={`form-control ${errors.name ? "is-invalid" : ""}`}
            {...register("name", { required: true, maxLength: 32 })}
            maxLength={32}
          />
          {errors.name && (
            <div className="invalid-feedback">{L("ThisFieldIsRequired")}</div>
          )}
        </div>

        <div className="mb-5">
          <label className="form-label required" htmlFor="surname">
            {L("Surname")}
          </label>
          <input
            id="surname"
            type="text"
            className={`form-control ${errors.surname ? "is-invalid" : ""}`}
            {...register("surname", { required: true, maxLength: 32 })}
            maxLength={32}
          />
          {errors.surname && (
            <div className="invalid-feedback">{L("ThisFieldIsRequired")}</div>
          )}
        </div>

        <div className="mb-5">
          <label className="form-label" htmlFor="emailAddress">
            {L("EmailAddress")}
          </label>
          <input
            id="emailAddress"
            type="email"
            className={`form-control ${
              errors.emailAddress ? "is-invalid" : ""
            }`}
            {...register("emailAddress", {
              maxLength: 255,
              pattern: {
                value: /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$/,
                message: L("InvalidEmailAddress"),
              },
            })}
            maxLength={255}
          />
          {errors.emailAddress && (
            <div className="invalid-feedback">
              {errors.emailAddress.message || L("InvalidEmailAddress")}
            </div>
          )}
        </div>
      </form>
    </Modal>
  );
};

export default CreatePersonModal;
```

Let me explain some parts of this component:

- It accepts three **props**: `isVisible` to control modal visibility, `onClose` to handle modal close, and `onSave` to notify parent component when person is saved.
- Uses **react-hook-form** for form management with real-time validation (`mode: "onChange"`).
- Uses **useServiceProxy** hook to get an instance of **PersonServiceProxy** to call server side methods.
- Uses **useState** hook for managing the `saving` state during API calls.
- Uses **useEffect** to reset form values when modal becomes visible.
- **Auto-focuses** the name input field when modal opens using `setFocus` from react-hook-form.
- Implements **form validation** with required fields and maxLength constraints.
- Uses **ABP notification** system (`abp.notify.success`) to show success messages.

Open PhoneBookDemo.xml (the **default**, **English** localization dictionary) and add the following line:

```xml
<text name="CreateNewPerson">Create New Person</text>
```

Now we need to integrate this modal into the phonebook page. Update **index.tsx** as shown below:

```typescript
import L from "@/lib/L";
import React, { useEffect, useState } from "react";
import { Table } from "antd";
import type { ColumnsType } from "antd/es/table";
import PageHeader from "../components/common/PageHeader";
import { useTheme } from "@/hooks/useTheme";
import {
  PersonListDto,
  PersonServiceProxy,
  useServiceProxy,
} from "@/api/service-proxy-factory";
import CreatePersonModal from "./CreatePersonModal";

const PhoneBookPage: React.FC = () => {
  const { containerClass } = useTheme();
  const personService = useServiceProxy(PersonServiceProxy);

  const [people, setPeople] = useState<PersonListDto[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  const [filter, setFilter] = useState<string>("");
  const [isCreateModalVisible, setIsCreateModalVisible] =
    useState<boolean>(false);

  useEffect(() => {
    getPeople();
  }, []);

  const getPeople = async (): Promise<void> => {
    setLoading(true);
    try {
      const result = await personService.getPeople(filter);
      setPeople(result.items || []);
    } finally {
      setLoading(false);
    }
  };

  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault();
    getPeople();
  };

  const handleCreatePerson = () => {
    setIsCreateModalVisible(true);
  };

  const handleModalClose = () => {
    setIsCreateModalVisible(false);
  };

  const handleModalSave = () => {
    getPeople();
  };

  const columns: ColumnsType<PersonListDto> = [
    {
      title: L("Name"),
      dataIndex: "name",
      key: "name",
      sorter: true,
      width: 150,
    },
    {
      title: L("Surname"),
      dataIndex: "surname",
      key: "surname",
      sorter: true,
      width: 150,
    },
    {
      title: L("EmailAddress"),
      dataIndex: "emailAddress",
      key: "emailAddress",
      sorter: true,
      width: 250,
    },
  ];

  return (
    <>
      <PageHeader
        title={L("PhoneBook")}
        description={L("PhoneBooksHeaderInfo")}
      />
      <div className={containerClass}>
        <div className="container-fluid">
          <div className="card card-custom gutter-b">
            <div className="card-header">
              <div className="card-toolbar ms-auto">
                <button
                  className="btn btn-primary"
                  type="button"
                  onClick={handleCreatePerson}
                >
                  <i className="fa fa-plus"></i> {L("CreateNewPerson")}
                </button>
              </div>
            </div>
            <div className="card-body">
              <form className="form" autoComplete="off" onSubmit={handleSearch}>
                <div className="row align-items-center mb-4">
                  <div className="col-xl-12">
                    <div className="mb-5 m-form__group align-items-center">
                      <div className="input-group">
                        <input
                          value={filter}
                          name="filter"
                          autoFocus
                          className="form-control m-input"
                          placeholder={L("SearchWithThreeDot")!}
                          type="text"
                          onChange={(e) => setFilter(e.target.value)}
                        />
                        <button className="btn btn-primary" type="submit">
                          <i
                            className="flaticon-search-1"
                            aria-label={L("Search")!}
                          ></i>
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              </form>

              <Table
                rowKey={(record) => record.id!}
                size="small"
                bordered
                columns={columns}
                loading={loading}
                dataSource={people}
                pagination={false}
                scroll={{ x: true }}
              />
            </div>
          </div>
        </div>
      </div>

      <CreatePersonModal
        isVisible={isCreateModalVisible}
        onClose={handleModalClose}
        onSave={handleModalSave}
      />
    </>
  );
};

export default PhoneBookPage;
```

Key changes made in the phonebook page:

- **Imported** the `CreatePersonModal` component
- Added **state** for modal visibility using `useState` hook: `isCreateModalVisible`
- Created handler functions:
  - `handleCreatePerson` - opens the modal
  - `handleModalClose` - closes the modal
  - `handleModalSave` - refreshes the people list after successful save
- Added a **"Create New Person"** button in the card header that triggers `handleCreatePerson`
- Placed the **CreatePersonModal** component at the bottom of the JSX, passing the required props

The modal is now fully integrated! When you click the "Create New Person" button, the modal will open. After successfully creating a person, the modal closes and the people list automatically refreshes to show the new entry.

## Next

- [Authorization For Phone Book](Developing-Step-By-Step-React-Authorization-PhoneBook)