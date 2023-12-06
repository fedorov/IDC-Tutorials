# Check Commits and Colab Images

This GitHub repository uses a GitHub Actions workflow to check if getting started notebooks in IDC-Tutorials are working as expected in the Google Colab environment.

# Status

[![Getting Started Notebooks in the latest Colab environment](https://github.com/ImagingDataCommons/IDC-Tutorials/actions/workflows/test_colab.yml/badge.svg)](https://github.com/ImagingDataCommons/IDC-Tutorials/actions/workflows/test_colab.yml)

## Workflow

1. **Check for Image Changes**:
   - Make an API call to Artifact Registry to check if there are new Docker images.
     
      ```shell
      gcloud artifacts docker tags list us-docker.pkg.dev/colab-images/public/runtime --format=json --quiet
      ```
   - Compare the `sh256digest` with the previous latest image.

2. **Preprocess Notebooks**:
   - Use an IDC Google Cloud Project ID, instead of getting it interactively.
   - Handle typical authentication from Colab notebooks using Application Default Credentials instead of `auth.authenticate_user()`.
   - The action `google-github-actions` when used with `export_environment_variables: true` exposes the path of Application Default Credentials with the env variable GOOGLE_APPLICATION_CREDENTIALS.
   - Some notebooks require the user to enter the query. In such cases, the expected query is induced.

3. **Docker Image Handling**:
   - If the Colab Docker image is changed, pull it and push it to Docker Hub (as the frequency of Colab image updates is shorter than the frequency of pulling the image for testing, we do not want to pile up charges by using Artifact Registry directly).
   - If no changes, just pull the image from Docker Hub.
   - To save disk space, use [`jlumbroso/free-disk-space@main`](https://github.com/jlumbroso/free-disk-space) to gain additional storage.

4. **Running Notebooks with Papermill**:
   - Attach the repository source directory to the container's `/content` folder.
   - Install the [`papermill`](https://papermill.readthedocs.io/) package to run the notebooks.
   - Capture `papermill` output and handle any errors.

5. **Update Repository**:
   - Automatically commit the output files generated by the Docker container using [`stefanzweifel/git-auto-commit-action@v4`](https://github.com/stefanzweifel/git-auto-commit-action).
   - Offers a quick way to see, at which cell the notebook failed.

## Prerequisites

Before using the workflow, make sure to set the required secrets in your repository:

- `SERVICE_ACCOUNT_KEY`: Google Cloud service account key JSON (make sure to convert it to ONE LINE JSON).
 Note: minimum permissions required for the service account: `Bigquery User`
- `DOCKER_USERNAME`: Docker Hub username.
- `DOCKER_PASSWORD`: Docker Hub password or access token.

## Resources

- [Papermill](https://papermill.readthedocs.io/)
- [Application Default Credentials based login](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)
- [Google GitHub Actions](https://github.com/google-github-actions)
- [Commits](https://github.com/vkt1414/track-colab-env/commits/main)
- [Space Saving](https://github.com/jlumbroso/free-disk-space)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.