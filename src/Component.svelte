<script>
  import { getContext } from "svelte";

  // export let text
  import "bootstrap/dist/css/bootstrap.min.css";
  import Poper from "popper.js";
  import "bootstrap/dist/js/bootstrap.min.js";

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  import {
    S3Client,
    PutObjectCommand,
    ListObjectsCommand,
  } from "@aws-sdk/client-s3";

  const ACCESS_KEY_ID = "DQU7J6TFV3HUMJ5RYS6F";
  const SECRET_ACCESS_KEY = "BkrPzsTEnpkeBZKCS9OBirYeCnSpz9VPNJebo0g0";
  const END_POINT = "https://s3.us-east-2.wasabisys.com";
  const BUCKET = "budibase-testing";

  const s3Client = new S3Client({
    region: "us-east-2",
    endpoint: END_POINT,
    credentials: {
      accessKeyId: ACCESS_KEY_ID,
      secretAccessKey: SECRET_ACCESS_KEY,
    },
  });

  async function listFiles() {
    const params = {
      Bucket: BUCKET,
    };

    try {
      // const listResponse = await s3Client.send(new AWS.ListObjectsCommand(params));
      const listResponse = await s3Client.send(new ListObjectsCommand(params));
      const files = listResponse.Contents;

      // Clear the table body first
      const tableBody = document.querySelector(".file-table tbody");
      tableBody.innerHTML = "";

      // Append each file's information to the table
      files.forEach((file, index) => {
        const row = document.createElement("tr");
        row.innerHTML = `
          <th scope="row">${index + 1}</th>
          <td>${file.Key}</td>
          <td>${file.Size} bytes</td>
          <td>${file.LastModified}</td>
        `;
        tableBody.appendChild(row);
      });

      console.log("Files in the bucket:", files);
    } catch (error) {
      console.error("Error listing files:", error);
    }
  }

  async function uploadFile() {
    const fileInput = document.getElementById("formFile");
    const file = fileInput.files[0];

    if (file) {
      const params = {
        Bucket: BUCKET,
        Key: file.name,
        Body: file,
      };

      try {
        // const uploadResponse = await s3Client.send(new AWS.PutObjectCommand(params));
        const uploadResponse = await s3Client.send(
          new PutObjectCommand(params)
        );
        alert("File Uploaded Successfully!");
        console.log("File uploaded successfully:", uploadResponse);
        listFiles();
      } catch (error) {
        console.log("Error uploading file:", error);
      }
    } else {
      alert("No file selected for upload.");
    }
  }
</script>

<div use:styleable={$component.styles}>
  <div class="container mt-3">
    <div class="row">
      <div class="col-md-8 mx-auto center-card">
        <div class="card shadow p-3">
          <div class="card-body">
            <h5 class="card-title">Upload File To Wasabi Bucket</h5>
            <div class="mb-3">
              <label for="formFile" class="form-label"
                >Select a File To Upload</label
              >
              <input class="form-control" type="file" id="formFile" />
            </div>

            <div class="btn-group">
              <div class="form-group">
                <button
                  on:click={uploadFile}
                  style="cursor: pointer;"
                  class="btn btn-outline-primary">Upload File</button
                >
              </div>

              <div style="margin-left:10px;" class="form-group">
                <button
                  on:click={listFiles}
                  style="cursor: pointer;"
                  class="btn btn-outline-primary">List Files</button
                >
              </div>
            </div>

            <div class="table-responsive">
              <table class="table table-hover mt-4 file-table">
                <thead>
                  <tr>
                    <th scope="col">#</th>
                    <th scope="col">File</th>
                    <th scope="col">Size</th>
                    <th scope="col">Last Modified</th>
                  </tr>
                </thead>
                <tbody />
              </table>
            </div>
            <!-- <p class="card-text">
                      
                    </p> -->
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
