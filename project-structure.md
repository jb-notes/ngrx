Building an enterprise-grade Angular project structure
[sample repo](https://github.com/Gbuomprisco/cryptofolio)

## Angular Entities
- modules
- components
- directives
- services
- pipe
- guards

Naming convention
- [component name].[entity type].ts
- If we create a pipe whose class is called HighlightPipe, we will name its file highlight.pipe.ts
- If we have a component called DropdownComponent we want to its files dropdown.component.ts, dropdown.component.html and dropdown.component.scss.
- Dollar sign for Observables, easily identify what is an Observable
  -  `public assets$ = this.store.select(selectAllAssets);`
  -  
###### References 
[https://angularbites.com/building-an-enterprise-grade-angular-project-structure/](https://angularbites.com/building-an-enterprise-grade-angular-project-structure/)

## Feature Modules
- As Angular apps are made of modules that can import other modules
- Each module will contain all other Angular entities contained in their own folders.
- A Feature Module is not supposed to export anything except the top component, so anything we define within it will not be used elsewhere.
  
![image](https://user-images.githubusercontent.com/21466657/96357471-e9ab9200-10c1-11eb-8dbb-707ef1663d24.png)

## Shared Modules
A SharedModule is usually made up of entities that are shared across different modules within a project — but aren’t normally needed outside of it.

When we do encounter services or components that can be reused across different teams and projects, and that ideally don’t change very often, we may want to build an Angular Library.

![image](https://user-images.githubusercontent.com/21466657/96357483-1d86b780-10c2-11eb-90ff-3f802835568c.png)

## Typescript Entities
Typescript entities :
  - classes
  - enums
  - interfaces (and types)

> I like to group these entities in their own folder within a module, which I reluctantly call core, but this is very much up to you and your team to decide. - @gc_psk

![image](https://user-images.githubusercontent.com/21466657/96357505-84a46c00-10c2-11eb-8190-ef6698ff18d4.png)

## Overview
![image](https://user-images.githubusercontent.com/21466657/96402607-f43a5a00-119b-11eb-9e35-13c40df5b101.png)
