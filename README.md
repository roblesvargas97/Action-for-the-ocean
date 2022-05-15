# Action for the ocean

Action for the ocean es un proyecto realizado para el hackathon de LaunchX que ayudara a la conservación de playas y oceanos de México. 


### Star Proyect

1. Clone the repository.
2. Create a new database in PostgreSQL.
3. Run the following commands in the terminal: `cd server`
4. Run the following commands in the terminal: `npm install`
5. Run the following commands in the terminal: `npx prisma migrate dev --name init` (for creating the table)
6. Create inside the server folder a file called `.env` with the following content: `DATABASE_URL="postgresql://USER:PASSWORD@localhost:5432/DATABASENAME?schema=public"` (replace USER, PASSWORD and DATABASENAME with your credentials)
7. Run the following commands in the terminal: `npm prisma/seed.js` (for enter the data in the database)
8. Run the following commands in the terminal: `npm start`
9. In another terminal, run the following commands: `cd client`
10. Run the following commands in the terminal: `npm install`
11. Run the following commands in the terminal: `npm start`
12. Visit the following URL: `http://localhost:3000`

## Server

Se levanta el server en express en el puerto `3000` donde pasaremos las rutas por medio del routerApi creado en `./routes`.

``` javascript
const express = require("express");
const routerApi = require("./routes");
const app = express();
const port = 3000;

app.use(express.json());

app.get("/", (req , res) => {
    res.send("Hello World!");
});

routerApi(app);

app.listen(port, () => {
    console.log(`app listening on port ${port}`);
});
```
## Routes 

### Api Routes

Se crean la conexion de api para las routes:

``` javascript
const express = require("express");
const usersRoutes = require("./usersRoutes");
const usersBeach = require("./usersBeach");

function routerApi(app) {
    const router = express.Router();
    app.use("/api/v1", router);
    router.use("/users", usersRoutes);
    router.use("/usersBeach", usersBeach);
}

module.exports = routerApi;
```

### userRoutes
Aqui se crea la conexion con controllers/UserController una vez calidando el correo nos arroja 200 si es correcto y sino un 400 y un mensaje de error

``` javascript
const express = require("express");
const UserController = require("../controllers/UserController");

const router = express.Router();

router.post("/", async (req, res) => {
    try {
        const userData = {
            email: req.body.email,
            password: req.body.password,
        };
        const login = await UserController.login(userData);
        return res.status(200).json({
            message:login
        });
    } catch (error) {
        res.status(404).json({
            message: error.message,
        });
    }
});

router.post("/register", async (req, res) => {
    try {
        const userData = {
            name: req.body.name,
            lastname: req.body.lastname,
            email: req.body.email,
            password: req.body.password,
            country: req.body.country
        };
        const register = await UserController.register(userData);
        return res.status(200).json({
            message: register
        });
    } catch (error) {
        res.status(400).json({
            message: error.message,
        });
    }
});

router.post("/update" , async (req , res) => {
    try {
        const userData = {
            email: req.body.email,
            password: req.body.password,
        };
        const userInfo = await UserController.getUserData(userData);
        return res.status(200).json({
            message: userInfo
        });
    } catch (error) {
        res.status(404).json({
            message: error.message,
        });
    }
});

router.put("/update", async (req, res) => {
    try {
        const userData = {
            name: req.body.name,
            lastname: req.body.lastname,
            email: req.body.email,
            password: req.body.password,
            country: req.body.country
        };
        const update = await UserController.updateUser(userData);
        return res.status(200).json({
            message: update
        });
    } catch (error) {
        res.status(400).json({
            message: error.message,
        });
    }
});

module.exports = router;
```

### userBeachRoutes

Aqui se crea la conexion con controllers/UserBeachController en caso de que la conexion sea correcta nos arrojara 200 en caso de que no sea correcta nos arroja 400 y un mensaje de error.

``` javascript
const express = require("express");
const UserBeachController = require("./../controllers/UserBeachController");

const router = express.Router();

router.post("/", async (req, res) => {
    try {
        const userData = {
            userId: req.body.userId,
            beachId: req.body.beachId,
            latitude: req.body.latitude,
            longitude: req.body.longitude,
            rating: req.body.rating
        };
        const createLocation = await UserBeachController.createLocation(userData);
        return res.status(200).json({
            message: createLocation
        });
    } catch (error) {
        res.status(400).json({
            message: error.message,
        });
    }
});

module.exports = router;
```

## Services
### UserServices

Aqui creamos los servcios del usuario donde se podra registrar, logear a la aplicacion y actualizar su información.

``` javascript 
    const { PrismaClient } = require("@prisma/client");
    const prisma = new PrismaClient();

    class UserService {
        static async register(user) {
            const message = "Usuario Creado";
            await prisma.user.create({ data: user });
            return message;
        }

        static async login(user) {
            const query = await prisma.user.findFirst({
                where: {
                    email: user.email,
                    password: user.password,
                },
            });
            if (query === null) {
                throw new Error("Usuario o Contraseña incorrecto");
            } else {
                const { id, name, email } = query;

                return {
                    id,
                    email,
                    name,
                };
            }
        }

        static async getUserData(user) {
            const query = await prisma.user.findFirst({
                where: {
                    email: user.email,
                },
            });
            if (query === null) {
                throw new Error("Usuario o Contraseña incorrecto");
            } else {
                const { name, lastname, email, password, country } = query;

                return {
                    name,
                    lastname,
                    email,
                    password,
                    country,
                };
            }
        }

        static async updateUser(user) {
            const query = await prisma.user.update({
                where: {
                    email: user.email,
                },
                data: {
                    name: user.name,
                    lastname: user.lastname,
                    email: user.email,
                    password: user.password,
                    country: user.country,
                },
            });

            return query;
        }
    }

    module.exports = UserService;
```
### UserBeach Services

Aqui en este servicio manejaremos la creacion de las locaciones y retornaremos las playas cercanas a la posicion del usuario, y traeremos el rating de las estrellas que tiene dicha playa.

``` javascript
const { PrismaClient } = require("@prisma/client");
const prisma = new PrismaClient();

class UserBeachService {

    static async createLoacation (location) {
        const message = "Locacion creada";
		
        const radius = 0.001;

        const latitude = location.latitude;
        const longitude = location.longitude;

        const higherLatitude = latitude + radius;
        const lowerLatitude = latitude - radius;
        const higherLongitude = longitude + radius;
        const lowerLongitude = longitude - radius;
		

        const beachId = await prisma.$queryRaw`select ID from (
		SELECT ID, (BEACH.latitude-${lowerLatitude}+${higherLatitude}-BEACH.latitude+ BEACH.longitude-${lowerLongitude}+${higherLongitude}-BEACH.longitude)/4 as promedio FROM BEACH
		WHERE BEACH.latitude between ${lowerLatitude} and ${higherLatitude}
		and BEACH.longitude between ${lowerLongitude} and ${higherLongitude}
		order by promedio asc
		)as subtable limit 1`;


		

        if(beachId){
            location.beachId = beachId;
        }else
            throw new Error("La locacion no se encuentra en una playa registrada"); 
		
        await prisma.userBeach.create({data: location});
        return message;
    }
	
    static async getAllRatings(){
        const ratings = await prisma.$queryRaw`select avg(userBeach.rating) as rating,beach.latitude, beach.longitude, beach.name from userBeach
		inner join beach on beach.id = userBeach.beachId group by beachId`;
		
        return ratings;
    }


}

module.exports = UserBeachService;
```



