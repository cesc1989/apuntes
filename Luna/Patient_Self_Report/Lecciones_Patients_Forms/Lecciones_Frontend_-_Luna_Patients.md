# Lecciones Frontend - Luna Patients

## Prevenir que yarn start o npm start arranque el navegador por defecto del sistema

Se puede hacer al inicializar la variable de ambiente BROWSER

Como explican en [Today I Learnt](https://til.hashrocket.com/posts/oejoedxwdf-custom-browser-with-create-react-app):


    {
      "name": "custom-browser",
      "version": "0.1.0",
      ...
      
      "scripts": {
        "start": "BROWSER='Google Chrome' react-scripts start",
        ...
      }
    }

O en [Stack Overflow](https://stackoverflow.com/questions/50686869/how-to-open-non-default-browsers-when-doing-npm-start-for-a-react-project):


    (export BROWSER=firefox;  npm start)

