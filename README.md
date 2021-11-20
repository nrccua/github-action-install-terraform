Enables the use of the `terraform` command line in your Github action. To use Terraform with AWS you will also need to set the AWS environment variables. See `Execute Terraform` step in the [Example](#Example)

# Usage

<!-- start usage -->
```yaml
- uses: nrccua/github-action-install-terraform
```
<!-- end usage -->

# Example
```yaml
name: Apply A Terraform Plan

on:
  push:
    branches:
      - master

jobs:
  test:
    name: Deploy A Build
    runs-on: ubuntu-latest
    env:
      aws_s3_bucket: ${{ secrets.AWS_S3_BUCKET }}
    steps:
      - uses: actions/checkout@v2
        with:
          persist-credentials: false
      - run: my_build.sh
      - name: Install Terraform
        uses: nrccua/github-action-install-terraform@master
      - name: Execute Terraform
        env:
          TF_VAR_aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          TF_VAR_aws_secret_key: ${{ secrets.AWS_SECRET_KEY_ID }}
          TF_VAR_aws_region: ${{ secrets.AWS_REGION }}
          TF_VAR_bucket_name: ${{ secrets.AWS_S3_BUCKET }}-pr-${{ github.event.number }}
        run: |
          terraform -chdir=./dir/to/tf_files init -input=false
          terraform -chdir=./dir/to/tf_files validate
          terraform -chdir=./dir/to/tf_files plan -out=tfplan -input=false
          terraform -chdir=./dir/to/tf_files apply -input=false tfplan
      - name: Install AWS CLI
        uses: nrccua/github-action-install-awscli@master
        with:
          aws_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_access_key: ${{ secrets.AWS_SECRET_KEY_ID }}
          aws_region: ${{ secrets.AWS_REGION }}
      - name: Copy files to S3
        run: |
          aws s3 cp --acl public-read --recursive ./my_build_directory s3://$aws_s3_bucket/
```
