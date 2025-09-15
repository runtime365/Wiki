

# Section 02 - Microservice Development

## Inhoudsopgave

- [8. Creating the first micro service](#8-creating-the-first-micro-service) - Project setup en solution structuur
- [9. Reviewing and simplifying the project](#9-reviewing-and-simplifying-the-project) - Template optimalisatie en middleware configuratie
- [10. Adding the entity classes](#10-adding-the-entity-classes) - Domain models en data structures

---

#### 8. Creating the first micro service

```bash
dotnet new sln

dotnet new webapi -o src/PostService -controllers

dotnet sln add src/PostService
```

We'll strip this down to the basics, starting with the most bare-bones version of a Web API template possible, then building up from there.


#### 9. Reviewing and simplifying the project

In a typical setup, we would include authentication and authorization middleware to inspect incoming requests and verify they meet our authorization requirements.
Each API controller has a designated route, with the routing middleware directing HTTP requests to the appropriate API endpoint.


#### 10. Adding the entity classes




























