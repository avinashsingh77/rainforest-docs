## 🐑️ Initialize Git Repositories, Login to JupyterHub
## Initialize a DAG repository for Airflow

1. Gitlab, create dag repo called **daintree-dev-dags**

2. In DevSpaces initialize dags repo.

    ```bash
    cd /projects
    git clone https://gitlab-ce.apps.foo-sno.sandbox1885.opentlc.com/rainforest/daintree-dev-dags.git
    cd daintree-dev-dags
    echo "# rainforest/daintree-dev-dags" > README.md
    git add README.md
    git commit -m "🦩 initial commit 🦩"
    git branch -M main
    git push -u origin main
    ```

## Clone code demo repo

3. Clone our demo code repo

   ```bash
   export GITHUB_TOKEN=<token>
   ```

   ```bash
   cd /projects/
   git clone https://$GITHUB_TOKEN@github.com/eformat/aiml-demos.git
   ```

4. Create a project in Gitlab called **aiml-demos** and push the code.

    ```bash
    cd /projects/aiml-demos
    git remote set-url origin https://${GITLAB_USER}:${GITLAB_PAT}@${GIT_SERVER}/${TEAM_NAME}/aiml-demos.git
    ```

   Use the `GITLAB_PAT` from above when you are prompted for the password (this will be cached)

    ```bash
    cd /projects/aiml-demos
    git add .
    git commit -am "🐙 ADD - aiml-demos 🐙"
    git push -u origin --all
    ```

## Login to JupyterHub

5. Login to daintree-dev project, browse to Jupyterhub route. Login as your <USER_NAME> using the **OpenShift v4** button and **FreeIPA** identity provider.

   ```bash
   echo -e https://$(oc get route jupyterhub --template='{{ .spec.host }}' -n ${PROJECT_NAME})
   ```

6. Select **Elyra Notebook Image**, Select **Small** container size, Set the following env vars, get values from DevSpaces

   ```bash
   echo $CLUSTER_DOMAIN
   echo $TEAM_NAME
   echo $PROJECT_NAME   
   echo $GITLAB_USER
   echo $GITLAB_PAT
   echo $GIT_SERVER
   ```
   
   ```bash
   AWS_ACCESS_KEY_ID=$(oc get secret s3-auth -n ${PROJECT_NAME} -o jsonpath='{.data.AWS_ACCESS_KEY_ID}' | base64 -d)
   AWS_SECRET_ACCESS_KEY=$(oc get secret s3-auth -n ${PROJECT_NAME} -o jsonpath='{.data.AWS_SECRET_ACCESS_KEY}' | base64 -d)
   ```

8. Click **Start server** button.

9. Once your notebook has opened, Click **Terminal** and clone the aiml-demos repo.

   ```bash
   git clone https://${GITLAB_USER}:${GITLAB_PAT}@${GIT_SERVER}/${TEAM_NAME}/aiml-demos.git
   ```

## Setup Elyra Airflow

9. Add the **airflow-runner** container image

   ```json
   mkdir -p ~/.local/share/jupyter/metadata/runtime-images/
   cat << EOF > ~/.local/share/jupyter/metadata/runtime-images/airflow-runner.json
   {
     "display_name": "airflow-runner",
     "metadata": {
       "tags": [],
       "display_name": "airflow-runner",
       "image_name": "image-registry.openshift-image-registry.svc:5000/rainforest-ci-cd/airflow-runner:2.3.2",
       "pull_policy": "Always"
     },
     "schema_name": "runtime-image"
   }
   EOF
   ```

10. Add the **ariflow** runtime configuration.

   ```json
   mkdir -p ~/.local/share/jupyter/metadata/runtimes
   cat << EOF >  ~/.local/share/jupyter/metadata/runtimes/airflow.json
   {
      "display_name": "airflow",
      "metadata": {
         "tags": [],
         "display_name": "airflow",
         "user_namespace": "daintree-dev",
         "git_type": "GITLAB",
         "github_api_endpoint": "https://${GIT_SERVER}",
         "api_endpoint": "https://airflow-daintree-dev.${CLUSTER_DOMAIN}",
         "github_repo": "rainforest/daintree-dev-dags",
         "github_branch": "main",
         "github_repo_token": "${GITLAB_PAT}",
         "cos_auth_type": "KUBERNETES_SECRET",
         "cos_endpoint": "http://minio.rainforest-ci-cd.svc.cluster.local:9000",
         "cos_bucket": "airflow-daintree-dev",
         "cos_secret": "s3-auth",
         "cos_username": "${AWS_ACCESS_KEY_ID}",
         "cos_password": "${AWS_SECRET_ACCESS_KEY}",
         "runtime_type": "APACHE_AIRFLOW"
      },
      "schema_name": "airflow"
   }
   EOF
   ```
