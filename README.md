# CodeCommit Repositories

The **CodeCommit Repositories** project contains a nested cloudformation stack which create the following:

* CodeCommit Repository
* An IAM Role with and inline policy which provides access to the created repository.
* An IAM Group with inline policy which allows member users to assume the created role.

The above is created as an atomic unit and can be created for any number of repositories by providing a **repository name** and **repository description**.

**NOTE**: Users are not created in this template and must be added by another means.

## Usage - Ops
**NOTE:** This is not as automated as it should be.  If we decide to take this further, we can operationalize all this more.



 Since we are using a nested stack, we must have an s3 bucket to upload the child stack templates.

> aws s3 mb --region ${region} --profile ${named\_profile} s3://${bucket\_name}


We have git ignore the **.cloudformation** so we can put our packaged template in there and do not accidently check it in.

> mkdir .cloudformation

Children templates need to reside in s3 so we **package** the templates next which will push templates to the s3 bucket we created as needed.  This will also modify our root template and replace local references to the child templates with references in S3.

> aws cloudformation package --region ${region} --profile ${named\_profile} --template ./src/cloudformation/main.yml --s3-bucket ${bucket\_name} --output json > .cloudformation/packaged-template.yml

From here we go to the console and create a new template with **.cloudformation/packaged-template.yml**

Once the template is done, we can add users to the creaed group(s)

## Usage - Developer

1. Create a set of access keys
2. Create a named profile with the access keys
3. Run the following commands

> aws configure set role_arn arn:aws:iam:::${account\_number}:role/${role\_name} --profile ${named\_profile\_for\_assumed\_role}

> aws configure set source_profile ${named_profile} --profile ${named\_profile\_for\_assumed\_role}

> aws configure set role_session_name "Richard_Bach" --profile ${named\_profile\_for\_assumed\_role}

> git clone codecommit::${Region}://${named\_profile\_for\_assumed\_role}@${Repository\_Name}