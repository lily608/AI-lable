import {
  GoogleGenAI,
  createUserContent,
  createPartFromUri,
} from "@google/genai";


const ai = new GoogleGenAI({ apiKey: "GOOGLE_API_KEY" });


async function main() {
  const myfile = await ai.files.upload({
    file: "path/to/sample.jpg",
    config: { mimeType: "image/jpeg" },
  });


  const response = await ai.models.generateContent({
    model: "gemini-2.0-flash",
    contents: createUserContent([
      createPartFromUri(myfile.uri, myfile.mimeType),
      "Caption this image.",
    ]),
  });
  console.log(response.text);
}


await main();
import com.google.api.gax.rpc.ApiException;
import com.google.cloud.dialogflow.cx.v3beta1.DetectIntentRequest;
import com.google.cloud.dialogflow.cx.v3beta1.DetectIntentResponse;
import com.google.cloud.dialogflow.cx.v3beta1.QueryInput;
import com.google.cloud.dialogflow.cx.v3beta1.QueryResult;
import com.google.cloud.dialogflow.cx.v3beta1.SessionName;
import com.google.cloud.dialogflow.cx.v3beta1.SessionsClient;
import com.google.cloud.dialogflow.cx.v3beta1.SessionsSettings;
import com.google.cloud.dialogflow.cx.v3beta1.TextInput;
import com.google.common.collect.Maps;
import java.io.IOException;
import java.util.List;
import java.util.Map;


public class DetectIntent {


  // DialogFlow API Detect Intent sample with text inputs.
  public static Map<String, QueryResult> detectIntent(
      String projectId,
      String locationId,
      String agentId,
      String sessionId,
      List<String> texts,
      String languageCode)
      throws IOException, ApiException {
    SessionsSettings.Builder sessionsSettingsBuilder = SessionsSettings.newBuilder();
    if (locationId.equals("global")) {
      sessionsSettingsBuilder.setEndpoint("dialogflow.googleapis.com:443");
    } else {
      sessionsSettingsBuilder.setEndpoint(locationId + "-dialogflow.googleapis.com:443");
    }
    SessionsSettings sessionsSettings = sessionsSettingsBuilder.build();


    Map<String, QueryResult> queryResults = Maps.newHashMap();
    // Instantiates a client.


    // Note: close() needs to be called on the SessionsClient object to clean up resources
    // such as threads. In the example below, try-with-resources is used,
    // which automatically calls close().
    try (SessionsClient sessionsClient = SessionsClient.create(sessionsSettings)) {
      // Set the session name using the projectID (my-project-id), locationID (global), agentID
      // (UUID), and sessionId (UUID).
      SessionName session =
          SessionName.ofProjectLocationAgentSessionName(projectId, locationId, agentId, sessionId);


      // TODO : Uncomment if you want to print session path
      // System.out.println("Session Path: " + session.toString());


      // Detect intents for each text input.
      for (String text : texts) {
        // Set the text (hello) for the query.
        TextInput.Builder textInput = TextInput.newBuilder().setText(text);


        // Build the query with the TextInput and language code (en-US).
        QueryInput queryInput =
            QueryInput.newBuilder().setText(textInput).setLanguageCode(languageCode).build();


        // Build the DetectIntentRequest with the SessionName and QueryInput.
        DetectIntentRequest request =
            DetectIntentRequest.newBuilder()
                .setSession(session.toString())
                .setQueryInput(queryInput)
                .build();


        // Performs the detect intent request.
        DetectIntentResponse response = sessionsClient.detectIntent(request);


        // Display the query result.
        QueryResult queryResult = response.getQueryResult();


        // TODO : Uncomment if you want to print queryResult
        // System.out.println("====================");
        // System.out.format("Query Text: '%s'\n", queryResult.getText());
        // System.out.format(
        //     "Detected Intent: %s (confidence: %f)\n",
        //     queryResult.getIntent().getDisplayName(),
        //         queryResult.getIntentDetectionConfidence());


        queryResults.put(text, queryResult);
      }
    }
    return queryResults;
  }
}
