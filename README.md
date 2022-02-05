# PDF Reader

In a nutshell PDF Reader it is a simple Go library for reading PDF files which enables text exctraction being in the form of `Plain Text` or `Formatted Text`. The very first developer of this library it was https://github.com/rsc/pdf, being forked and improved by https://github.com/ledongthuc/pdf. Cloudresty has forked ledongthuc's library with the aim to maintain and improve it further.

&nbsp;
__Features__
  - Get plain text content (without format)
  - Get content (including all font and formatting information)

&nbsp;
## Install

`go get -u github.com/cloudresty/pdf`

&nbsp;
## Examples
&nbsp;
### Read plain text

```golang
package main

import (
	"bytes"
	"fmt"

	"github.com/cloudresty/pdf"
)

//
// Main Function
//

func main() {

	pdf.DebugOn = true

	//
	// Read PDF File
	//

	pdfText, err := readPDF("file.pdf")

	if err != nil {
		panic(err)
	}

	fmt.Println(pdfText)
	
	return

}

//
// Read PDF Function
//

func readPDF(path string) (string, error) {

	//
	// Open PDF File
	//

	f, r, errPDFOpen := pdf.Open(path)
	
    defer f.Close()

	if errPDFOpen != nil {
		return "", errPDFOpen
	}

	//
	// Extract Plain Text
	//

	var buf bytes.Buffer

    b, errGetPlainText := r.GetPlainText()

	if errGetPlainText != nil {
        return "", errGetPlainText
    }

	buf.ReadFrom(b)

	return buf.String(), nil

}
```

&nbsp;
### Read all text with styles from PDF

```golang
func readPDF(path string) (string, error) {

	//
	// Open PDF File
	//

	f, r, errPDFOpen := pdf.Open(path)

	defer f.Close()

	if errPDFOpen != nil {
		return "", errPDFOpen
	}

	//
	// Extract Formatted Text
	//
	
	totalPage := r.NumPage()

	for pageIndex := 1; pageIndex <= totalPage; pageIndex++ {

		p := r.Page(pageIndex)

		if p.V.IsNull() {
			continue
		}

		var lastTextStyle pdf.Text
		texts := p.Content().Text

		for _, text := range texts {

			if isSameSentence(text, lastTextStyle) {

				lastTextStyle.S = lastTextStyle.S + text.S
		
			} else {

				fmt.Printf("Font: %s, Font-size: %f, x: %f, y: %f, content: %s \n", lastTextStyle.Font, lastTextStyle.FontSize, lastTextStyle.X, lastTextStyle.Y, lastTextStyle.S)
				lastTextStyle = text

			}

		}

	}

	return "", nil

}
```

&nbsp;
### Read text grouped by rows

```golang
package main

import (
	"fmt"
	"os"

	"github.com/cloudresty/pdf"
)

//
// Main Function
//

func main() {

	//
	// Read Local PDF File
	//

	content, errReadPDF := readPDF(os.Args[1])

	if errReadPDF != nil {
		panic(errReadPDF)
	}

	fmt.Println(content)

	return

}

func readPDF(path string) (string, error) {

	//
	// Open PDF File
	//

	f, r, errReadPDF := pdf.Open(path)

	defer func() {
		_ = f.Close()
	}()

	if errReadPDF != nil {
		return "", errReadPDF
	}

	//
	// Page Count
	//

	totalPage := r.NumPage()

	//
	// Loop Through Pages
	//

	for pageIndex := 1; pageIndex <= totalPage; pageIndex++ {

		p := r.Page(pageIndex)

		if p.V.IsNull() {
			continue
		}

		//
		// Get Rows
		//

		rows, _ := p.GetTextByRow()

		for _, row := range rows {

			println("Row: ", row.Position)

			for _, word := range row.Content {

				fmt.Println(word.S)

			}

		}

	}

	return "", nil
}
```
