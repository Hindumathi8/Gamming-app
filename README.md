# Gamming-app
Create a gaming application by using web development languages.
https://e798d55d-39ea-4ce9-814e-ca2b4c9718e9-00-dyiqwffu46o1.spock.replit.dev
client/src/App.tsx
/.gitignore
modules = ["nodejs-20", "bash", "web"]
run = "npm run dev"
hidden = [".config", ".git", "generated-icon.png", "node_modules", "dist"]

[nix]
channel = "stable-24_05"

[deployment]
deploymentTarget = "autoscale"
build = ["npm", "run", "build"]
run = ["npm", "run", "start"]

[[ports]]
localPort = 5000
externalPort = 80

[workflows]
runButton = "Project"

[[workflows.workflow]]
name = "Project"
mode = "parallel"
author = "agent"

[[workflows.workflow.tasks]]
task = "workflow.run"
args = "Start Game"

[[workflows.workflow]]
name = "Start Game"
author = "agent"

[[workflows.workflow.tasks]]
task = "shell.exec"
args = "npm run dev"
waitForPort = 5000
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
{"metadata":{"type":"font","boundingBox":{"xMin":0,"yMin":-260,"xMax":952,"yMax":739.234375},"generator":"Fontello export by Glueckkanja AG","format":"json","copyright":""},"cssFontWeight":"normal","cssFontStyle":"normal","glyphs":{"A":{"ha":698,"x_min":0,"x_max":698,"o":"m 698 0 l 397 0 l 375 124 l 122 124 l 103 0 l 0 0 l 249 730 l 251 730 l 698 0 z m 245 569 l 249 461 l 340 221 l 155 221 l 245 569 z"}},"familyName":"Inter","ascender":739,"descender":-260,"underlinePosition":-100,"underlineThickness":50,"boundingBox":{"yMin":-260,"xMin":0,"yMax":739.234375,"xMax":952},"resolution":1000,"original_font_information":{"format":0,"copyright":"","fontFamily":"Inter","fontSubfamily":"Regular","uniqueID":"","fullName":"Inter Regular","version":"","postScriptName":"Inter-Regular"}}
asset" : {
        "generator" : "Khronos glTF Blender I/O v1.8.19",
        "version" : "2.0"
    },
    "scene" : 0,
    "scenes" : [
        {
            "name" : "Scene",
            "nodes" : [
                0
            ]
        }
    ],
    "nodes" : [
        {
            "mesh" : 0,
            "name" : "Heart",
            "rotation" : [
                0,
                0,
                -0.7071068286895752,
                0.7071068286895752
            ],
            "translation" : [
                0,
                0.6625411510467529,
                0
            ]
        }
    ],
    "materials" : [
        {
            "doubleSided" : true,
            "name" : "Heart",
            "pbrMetallicRoughness" : {
                "baseColorTexture" : {
                    "index" : 0
                },
                "metallicRoughnessTexture" : {
                    "index" : 1
                },
                "roughnessFactor" : 0
            }
        }
    ],
    "meshes" : [
        {
            "name" : "Heart",
            "primitives" : [
                {
                    "attributes" : {
                        "POSITION" : 0,
                        "NORMAL" : 1,
                        "TEXCOORD_0" : 2
                    },
                    "indices" : 3,
                    "material" : 0
                }
            ]
        }
    ],
    "textures" : [
        {
            "sampler" : 0,
            "source" : 0
        },
        {
            "sampler" : 0,
            "source" : 1
        }
    ],
    "images" : [
        {
            "bufferView" : 4,
            "mimeType" : "image/png",
            "name" : "Image_0"
        },
        {
            "bufferView" : 5,
            "mimeType" : "image/png",
            "name" : "Image_1"
        }
    ],
    "accessors" : [
        {
            "bufferView" : 0,
            "componentType" : 5126,
            "count" : 538,
            "max" : [
                0.662283718585968,
                0.6605305075645447,
                0.4106346368789673
            ],
            import express, { type Request, Response, NextFunction } from "express";
import { registerRoutes } from "./routes";
import { setupVite, serveStatic, log } from "./vite";

const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.use((req, res, next) => {
  const start = Date.now();
  const path = req.path;
  let capturedJsonResponse: Record<string, any> | undefined = undefined;

  const originalResJson = res.json;
  res.json = function (bodyJson, ...args) {
    capturedJsonResponse = bodyJson;
    return originalResJson.apply(res, [bodyJson, ...args]);
  };

  res.on("finish", () => {
    const duration = Date.now() - start;
    if (path.startsWith("/api")) {
      let logLine = `${req.method} ${path} ${res.statusCode} in ${duration}ms`;
      if (capturedJsonResponse) {
        logLine += ` :: ${JSON.stringify(capturedJsonResponse)}`;
      }

      if (logLine.length > 80) {
        logLine = logLine.slice(0, 79) + "…";
      }

      log(logLine);
    }
  });

  next();
});

(async () => {
  const server = await registerRoutes(app);

  app.use((err: any, _req: Request, res: Response, _next: NextFunction) => {
    const status = err.status || err.statusCode || 500;
    const message = err.message || "Internal Server Error";

    res.status(status).json({ message });
    throw err;
  });

  // importantly only setup vite in development and after
  // setting up all the other routes so the catch-all route
  // doesn't interfere with the other routes
  if (app.get("env") === "development") {
    await setupVite(app, server);
  } else {
    serveStatic(app);
  }

  // ALWAYS serve the app on port 5000
  // this serves both the API and the client
  const port = 5000;
  server.listen({
    port,
    host: "0.0.0.0",
    reusePort: true,
  }, () => {
    log(`serving on port ${port}`);
  });
})();
import type { Express } from "express";
import { createServer, type Server } from "http";
import { storage } from "./storage";

export async function registerRoutes(app: Express): Promise<Server> {
  // put application routes here
  // prefix all routes with /api

  // use storage to perform CRUD operations on the storage interface
  // e.g. storage.insertUser(user) or storage.getUserByUsername(username)

  const httpServer = createServer(app);

  return httpServer;
}
import express, { type Request, Response, NextFunction } from "express";
import { registerRoutes } from "./routes";
import { setupVite, serveStatic, log } from "./vite";

const app = express();
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
import { users, type User, type InsertUser } from "@shared/schema";

// modify the interface with any CRUD methods
// you might need

export interface IStorage {
  getUser(id: number): Promise<User | undefined>;
  getUserByUsername(username: string): Promise<User | undefined>;
  createUser(user: InsertUser): Promise<User>;
}

export class MemStorage implements IStorage {
  private users: Map<number, User>;
  currentId: number;

  constructor() {
    this.users = new Map();
    this.currentId = 1;
  }
  sertUser): Promise<User>;
}

export class MemStorage implements IStorage {
  private users: Map<number, User>;
  currentId: number;

  constructor() {
    this.users = new Map();
    this.currentId = 1;
  }

  async getUser(id: number): Promise<User | undefined> {
    return this.users.get(id);
  }

  async getUserByUsername(username: string): Promise<User | undefined> {
    return Array.from(this.users.values()).find(
      (user) => user.username === username,
    );
  }

  async createUser(insertUser: InsertUser): Promise<User> {
    const id = this.currentId++;
    const user: User = { ...insertUser, id };
    this.users.set(id, user);
    return user;
  }
}

export const storage = new MemStorage();
server/vite.tsserver/vite.ts
shared/schema.ts
/tailwind.config.ts
/tsconfig.json
/vite.config.ts
