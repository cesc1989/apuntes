# Uso de 1Password CLI o Web

## Pasos para iniciar sesión en 1password con el CLI para Luna

- correr `eval $(op signin)`
- entrar host [https://team-lunacare.1password.com](https://team-lunacare.1password.com)
- mi correo: [francisco.quintero@ideaware.co](mailto:francisco.quintero@ideaware.co)
- la secret key que está en el perfil: xxx
- el password que está en Bitwarden
- el código de 6 digitos que está en Bitwarden auth

## Bajar las ENVs desde 1password

Correr comando:
```bash
luna app get-env-local
```

docs: [https://github.com/lunacare/luna-cli/?tab=readme-ov-file#app-env-var-setup](https://github.com/lunacare/luna-cli/?tab=readme-ov-file#app-env-var-setup)