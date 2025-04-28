
# Project Modification Guide

This guide provides instructions on how to modify and extend the project.

## Modifying Colors and Styles

The `frontend` folder contains an index.css file CSS (shown in the filepath below) with several different CSS variables throughout the file that can be changed at will, with the most integral ones to the UI at the top of the file, shown below. 

Two configurations exist; one for light mode, and one for dark mode; changing the variables within first `:root` will change colours for light mode, while variables within the `:root` under `@media (prefers-color-scheme: dark)` will alter the appearence of the dark mode of the app.

*Note: the UI theme is synced to your OS theme; if your system is it light mode, the website will automatically be in light mode, and vice versa.*

```css
/* Filepath: ./frontend/src/index.css */
:root {
  --text: #050315;
  --text-secondary: #6e6e6e;
  --text-button: #f0f0f0;
  --background: #ffffff;
  --background2: #f5f5f5;
  --background3: #d4dbdf;
  --header: #ffffff;
  --header-text: --primary;
  --primary: #111835;
  --secondary: oklch(from var(--primary) calc(l * 1.5) c h);;
  --accent: #1c187a;
  --bot-text: --background;
  --sender-text: #f0f0f0;
  --input: #c4c4c4;
  --placeholder-text: #696873;
  --green-text: #2e9832;
  --border: #cccccc;
  --feedback: #3b58ff;

  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;

  color-scheme: light dark;
  color: #213547;
  background-color: #ffffff;
  

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

@media (prefers-color-scheme: dark) {
  :root {
    --text: #ffffff;
    --text-secondary: #aaaaaa;
    --green-text: #65ff6a;
    --text-button: #0e0e0e;
    --background: #2c2c2c;
    --background2: #171717;
    --background3: #444444;
    --header: #171717;
    --header-text: #ffffff;
    --primary: #537ece;
    --secondary: #466ec4;
    --accent: #3949ff;
    --bot-text: --background;
    --sender-text: #424141;
    --input: #5c5c5c;
    --placeholder-text: #8e8e8e;
    --border: #5c5c5c;
    --feedback: #769aff;

    color: var(--text);
    background-color: var(--background);
  }
```

## Extending the API

### Adding New Endpoints

1. **Implement the Lambda Function**: Create a new Lambda function in the `lambda` folder within the `cdk` directory. This function should handle the logic of the new endpoint.
2. **Edit the Api Stack**: In cdk/lib/api-gateway-stack.ts add the lambda function and override the logical id
3. **Define the Endpoint**: Update the `OpenAPI_Swagger_Definition.yaml` file with the new endpoint's specifications, request parameters, and responses. Use the new logical id for the `x-amazon-apigateway-integration:
uri:
  Fn::Sub:` field. Also ensure that the httpMethod field is POST even if the endpoint uses a different http method.
4. **Deploy**: Use AWS CDK to deploy changes to your infrastructure. Confirm that the new endpoint and Lambda function are correctly set up in your environment.


## Modifying Frontend Text, Icons, and Logo

1. **Locate Components**:

   - For the main application UI, update components in `frontend/src/components`.
   - For the main pages, update the files in `frontend/src/pages`.

2. **Modify Logo**: To change the logo used throughout the app, navigate to the `frontend/public/` folder, and replace the `logo_dark.svg` AND the `logo_light.svg` files respectively. *If you do not replace both (ideally with one made for light mode, and one made for dark mode) the logo will not update for certain system themes.*

3. **Modify App Icon**: If you wish to change the app icon shown in your browser tabs when the page is open, navigate to `frontend/public/` and replace `icon.svg` with whichever picture file you desire.

4. **Modify Text and UI Icons**: Update specific text and icon configurations in each component file. Each component has its unique structure, so locate the relevant text or icon section within the component and make your changes.

For example, to alter the interview assistant page, modify `frontend/src/pages/CasePage/InterviewAssistant.jsx`.

After making the required changes in the fork created in the [Deployment Guide](./docs/deploymentGuide.md), the amplify deployment should automatically redeploy.

## Modifying the LLM

- **Change the model used in the application**:
  - Find `bedrockLLMParameter` in api-gateway-stack.ts
  - Change stringValue to the model ID of the model you would like to use. A list of thee available models and their IDs are listed [here](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)
    For example to change the model to meta llama3 8b instruct, change `bedrockLLMParameter` to:
  ```typescript
  const bedrockLLMParameter = new ssm.StringParameter(
    this,
    "BedrockLLMParameter",
    {

      parameterName: "/LAT/BedrockLLMId",

      description: "Parameter containing the Bedrock LLM ID",
      stringValue: "meta.llama3-8b-instruct-v1:0",
    }
  );
  ```
  - Add permissions to invoke the model selected by finding `bedrockPolicyStatement` and changing the model id. 
  For example to change the model to meta llama3 8b instruct, change `bedrockPolicyStatement` to:
  ```typescript
  const bedrockPolicyStatement = new iam.PolicyStatement({
    effect: iam.Effect.ALLOW,
    actions: ["bedrock:InvokeModel", "bedrock:InvokeEndpoint"],
    resources: [
      "arn:aws:bedrock:" +
        this.region +
        "::foundation-model/meta.llama3-8b-instruct-v1:0",
      "arn:aws:bedrock:" +
        this.region +
        "::foundation-model/amazon.titan-embed-text-v2:0",
    ],
  });
  ```
  - redeploy the application by using the cdk deploy command in the deployment guide.

  - The `system_prompt` in `cdk/text_generation/src/helpers/chat.py` may require updates when switching models. *Note: temporarily, this can also be changed without a redeployment for debugging purposes from the admin page on the deployed app; simply navigate to the "AI Settings" page and alter the system prompt from the UI. However this will only change the prompt for the specific deployment you alter it from.*

