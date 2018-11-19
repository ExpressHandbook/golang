# How to Unzip a File in Golang by Reading from HTTP Response

Here is the code to unzip contents read from HTTP response.body using standard golang pacakges

```
package main

import (
	"archive/zip"
	"bytes"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"path/filepath"
	"time"
)

func main() {
	// Download zip
	downloadURL := "https://downloads.wordpress.org/plugin/sitewit-engagement-analytics.2.6.0.zip"
	client := http.Client{
		Timeout: time.Duration(5 * time.Second),
	}

	resp, err := client.Get(downloadURL)
	if err != nil {
		log.Fatal("Error downloading zip", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != 200 {
		log.Fatal("Error downloading zip.  Returned status code", resp.StatusCode)
	}

	zipContent, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal("Error reading zip contents", err)
	}

	zipReader, err := zip.NewReader(bytes.NewReader(zipContent), int64(len(zipContent)))
	if err != nil {
		log.Fatal("Error creating new zip reader", err)
	}

	destination := "/tmp/"
	for _, file := range zipReader.File {
		fileH, err := file.Open()
		if err != nil {
			log.Fatal("Error to  open file", err)
		}
		fpath := destination + "wp-content/plugins/" + file.Name
		if file.FileInfo().IsDir() {
			err = os.MkdirAll(fpath, 0755)
			if err != nil {
				log.Fatal("Error creating directory", fpath)
			}
		} else {
			if err = os.MkdirAll(filepath.Dir(fpath), 0755); err != nil {
				log.Fatal("Error creating directory", filepath.Dir(fpath))
			}

			outFile, err := os.OpenFile(fpath, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, file.Mode())
			if err != nil {
				log.Fatal("Error to open file", fpath)
			}

			_, err = io.Copy(outFile, fileH)
			if err != nil {
				log.Fatal("Error writing file", fpath)
			}
			// Close the file without defer to close before next iteration of loop
			outFile.Close()
		}
	}
}

```
