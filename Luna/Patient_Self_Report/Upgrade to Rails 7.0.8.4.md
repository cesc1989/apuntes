# Upgrade a Rails 7.0.8.4

## Error de OpenStruct por WickedPDF

Este error cuando corría las pruebas:
```bash
Failure/Error: OpenStruct.new(valid?: true, errors: @errors)

      NameError:
        uninitialized constant Answers::ValidateValue::OpenStruct

                OpenStruct.new(valid?: true, errors: @errors)
                ^^^^^^^^^^
```