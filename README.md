# File API
A simple RESTful API that allows you to manipulate files and directories.

> [!WARNING]
> This project is in early development stage.

> [!IMPORTANT]
> 💀 There are no robust protections against malicious users. This API is intended to be used as an internal API for simple projects. Usage of a service mesh that encrypts traffic between services is recommended.

# Usage

## Running in Docker

The easiest way to run the File API is to use the Docker image. The image is available on [Docker Hub](https://hub.docker.com/r/xkonti/file-api):

```bash
docker run -d -p 3210:3000 -e API_KEY=YOUR_API_KEY -v /your/directory/to/mount:/data file-api:latest
```

Available configuration options:
- Internal port: `3000`
- Mount point for the directory to be managed via API: `/data`
- `API_KEY` environment variable: the API key to use for authentication

## Authentication

An API key must be configured in order to use the API. The API key can be configured using the `API_KEY` environment variable. Please use something that is base64 encoded so that it can be used in HTTP headers and URL query.

Each HTTP request will be first checked for API key. If the API key is not provided, the request will be rejected with a `401 Unauthorized` response. The API key can be provided in two ways:
- HTTP header: `api-key` - example: `curl --request GET --url 'http://localhost:3000/list?path=%2F' --header 'apikey: YOUR_API_KEY'`
- Query parameter: `apikey` - example: `curl --request GET --url 'http://localhost:3000/list?path=%2F&apikey=YOUR_API_KEY'`


## API

### List files and directories in a directory

You can list files and/or directories by using the `/list` endpoint with the following parameters:
- `path` - the path of the directory to list (remember to encode the path)
- `dirs` - whether to include directories or not (default: `false`). If directories are listed their contents will be nested.
- `depth` - how deep to list directories. The default `1` will list only files and directories within the specified directory -  no contents of subdirectories will be listed.

<details>
  <summary>Examples</summary>

- `curl --request GET --url 'http://localhost:3000/list?path=%2F'` - list only files in the root directory (`%2F` is the encoded `/`):
  ```json
  [
    {
      "name": "Expenses 2022.xlsx",
      "fullPath": "/Expenses 2022.xlsx",
      "type": "file"
    }
  ]
  ```
- `curl --request GET --url 'http://localhost:3000/list?path=%2F&dirs=true'` - list files and directories in the root directory (`%2F` is the encoded `/`):
  ```json
  [
    {
      "name": "repos",
      "fullPath": "/repos",
      "type": "dir"
    },
    {
      "name": "Expenses 2022.xlsx",
      "fullPath": "/Expenses 2022.xlsx",
      "type": "file"
    }
  ]
  ```

- `curl --request GET --url 'http://localhost:3000/list?path=%2F&dirs=true&depth=2'` - list files and directories in the root directory. Additionally increase depth to `2` so that the contents of the `repos` directory are listed:
  ```json
  [
    {
      "name": "repos",
      "fullPath": "/repos",
      "type": "dir",
      "contents": [
        {
          "name": "rusty-result-ts",
          "fullPath": "/repos/rusty-result-ts",
          "type": "dir"
        },
        {
          "name": "vue-smart-routes",
          "fullPath": "/repos/vue-smart-routes",
          "type": "dir"
        },
        {
          "name": "todo.md",
          "fullPath": "/repos/todo.md",
          "type": "file"
        },
        {
          "name": "clean-node-package-typescript",
          "fullPath": "/repos/clean-node-package-typescript",
          "type": "dir"
        },
        {
          "name": "notes.md",
          "fullPath": "/repos/notes.md",
          "type": "file"
        }
      ]
    },
    {
      "name": "Expenses 2022.xlsx",
      "fullPath": "/Expenses 2022.xlsx",
      "type": "file"
    }
  ]
  ```
- `curl --request GET --url 'http://localhost:3000/list?path=%2F&dirs=true&depth=2'` - list **only files** in the root directory. Additionally increase depth to `2` so that the contents of the `repos` directory are listed. This will present all the files as a flat list:
  ```json
  [
    {
      "name": "todo.md",
      "fullPath": "/repos/todo.md",
      "type": "file"
    },
    {
      "name": "notes.md",
      "fullPath": "/repos/notes.md",
      "type": "file"
    },
    {
      "name": "Expenses 2022.xlsx",
      "fullPath": "/Expenses 2022.xlsx",
      "type": "file"
    }
  ]
  ```
- `curl --request GET --url 'http://localhost:3000/list?path=repos%2Frusty-result.ts&dirs=true'` - list files and directories in the `repos/rusty-result.ts` directory. Additionally increase depth to `2` so that the contents of the `repos` directory are listed:
  ```json
  [
    {
      "name": "repos",
      "fullPath": "/repos",
      "type": "dir",
      "contents": [
        {
          "name": "rusty-result-ts",
          "fullPath": "/repos/rusty-result-ts",
          "type": "dir"
        },
        {
          "name": "vue-smart-routes",
          "fullPath": "/repos/vue-smart-routes",
          "type": "dir"
        },
        {
          "name": "todo.md",
          "fullPath": "/repos/todo.md",
          "type": "file"
        },
        {
          "name": "clean-node-package-typescript",
          "fullPath": "/repos/clean-node-package-typescript",
          "type": "dir"
        },
        {
          "name": "notes.md",
          "fullPath": "/repos/notes.md",
          "type": "file"
        }
      ]
    },
    {
      "name": "Expenses 2022.xlsx",
      "fullPath": "/Expenses 2022.xlsx",
      "type": "file"
    }
  ]
  ```
</details>

## Roadmap

### Directories

- [x] List files and directories in a directory
- [ ] Check if a directory exists
- [ ] Create a directory
- [ ] Delete a directory
- [ ] Rename a directory
- [ ] Move a directory
- [ ] Copy a directory

### Files

- [ ] Check if a file exists
- [ ] Download a file
- [ ] Upload a file
- [ ] Delete a file
- [ ] Create an empty file
- [ ] Rename a file
- [ ] Move a file
- [ ] Copy a file
- [ ] Get file metadata

### Permissions

- [x] API key authentication
- [ ] Protect paths with API keys
- [ ] Protect operation categories with API keys
    - [ ] Listing files and directories
    - [ ] Downloading files
    - [ ] Uploading/copying files & creating directories
    - [ ] Deleting/moving/renaming files & directories

### Deployment

- [x] Docker image on Docker Hub
- [ ] Docker image on GitHub Container Registry
- [ ] CI/CD pipeline using GitHub Actions and Dagger

## Stack

- Runtime: Bun
- Bundler: Bun
- API framework: Elysia.js