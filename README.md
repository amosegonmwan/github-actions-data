# React NodeJS Project with GitHub Actions

This is a React NodeJS project that utilizes GitHub Actions for continuous integration and deployment. The project is structured to run tests, build the website, and deploy it when changes are pushed to the main branch.

## Workflow 1: Deploy website

```yaml
name: Deploy website
on:
  push:
    branches:
      - main
jobs:

  # Test job runs linting and tests
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Lint code
        run: npm run lint
      - name: Test code
        run: npm run test

  # Build job depends on the success of the test job
  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      script-file: ${{ steps.publish.outputs.script-file }}
    steps:
      - name: Get code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
      - name: Publish JS filename
        id: publish
        run: find dist/assets/*.js -type f -execdir echo 'script-file={}' >> $GITHUB_OUTPUT ';'
      - name: Upload Artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist-files
          path: dist

  # Deploy job depends on the success of the build job
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Get build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist-files
      - name: Output contents
        run: ls
      - name: Output filename
        run: echo "${{ needs.build.outputs.script-file }}"
      - name: Deploy
        run: echo "Deploying..."
```


## Workflow Details:

### Test Job:
* The test job runs on an Ubuntu environment.
* It checks out the code using actions/checkout@v3.
* Caches npm dependencies to improve build times.
* Installs project dependencies using npm ci.
* Lints the code using npm run lint.
* Runs tests using npm run test.

### Build Job:
* The build job depends on the success of the test job.
* It runs on an Ubuntu environment.
* Caches npm dependencies for faster builds.
* Installs project dependencies.
* Builds the website using npm run build.
* Publishes the JS filenames as artifacts for later use.
* Uploads build artifacts to be used by the deploy job.

### Deploy Job:
* The deploy job depends on the success of the build job.
* It runs on an Ubuntu environment.
* Downloads build artifacts from the build job.
* Outputs the contents of the deployed files.
* Outputs the filename of the published JS file.
* Echoes "Deploying..." to simulate the deployment process.
