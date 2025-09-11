# PDFAI Bundle for Symfony
[PDFAI](https://github.com/1tomany/pdf-ai) is a simple PHP library that makes extracting data from PDFs for large language models easy.

## Install PDFAI 
```shell
composer require 1tomany/pdf-ai-bundle
```

## Usage
Symfony will autowire the necessary classes after the bundle is installed.

```php
<?php

namespace App\Files;

use OneToMany\PDFAI\Contract\Action\ExtractDataActionInterface;
use OneToMany\PDFAI\Contract\Action\ReadMetadataActionInterface;
use OneToMany\PDFAI\Request\ExtractDataRequest;
use OneToMany\PDFAI\Request\ExtractTextRequest;
use OneToMany\PDFAI\Request\ReadMetadataRequest;

final readonly class FileHandler
{
    public function __construct(
        private ReadMetadataActionInterface $readMetadataAction,
        private ExtractDataActionInterface $extractDataAction,
    ) {
    }

    public function handle(string $filePath): void
    {
        // Read PDF metadata like page count
        $metadata = $this->readMetadataAction->act(new ReadMetadataRequest($filePath));

        // Rasterize all pages of a PDF to a 150 DPI PNG
        $request = new ExtractDataRequest(
            $filePath,       // Full path to PDF file
            1,               // First page to extract
            null,            // Last page to extract, NULL for all pages
            OutputType::Png, // Jpg and Txt are other options
            150,             // Output resolution in dots per inch
        );
        
        // @see OneToMany\PDFAI\Contract\Response\ExtractedDataResponseInterface
        foreach ($this->extractDataAction->act($request) as $image) {
            // $image->getData() or $image->toDataUri()
        }
        
        // Extract text from pages 2 through 8
        $request = new ExtractTextRequest($filePath, 2, 8);
        
        // @see OneToMany\PDFAI\Contract\Response\ExtractedDataResponseInterface
        foreach ($this->extractDataAction->act($request) as $text) {
            // $text->getData() or $text->toDataUri()
        }
    }
}
```

### Testing
If you wish to avoid interacting with an external process in your test environment, you can take advantage of the `MockExtractorClient` by simply setting the `1tomany.pdfai_extractor_client` parameter to the value `mock` in your Symfony service configuration for the `test` environment.

```yaml
when@test:
    parameters:
        1tomany.pdfai_extractor_client: 'mock'
```

Without changing _any_ other code, Symfony will automatically inject the `MockExtractorClient` instead of the default `PopplerExtractorClient` for your tests.

### Extending
Don't want to use Poppler? No problem! Create your own extractor class that implements the `OneToMany\PDFAI\Contract\Client\ExtractorClientInterface` interface and tag it accordingly:

```yaml
parameters:
    1tomany.pdfai_extractor_client: 'imagemagick'

services:
    App\File\Extractor\Client\ImageMagickExtractorClient:
        tags:
            - { name: 1tomany.pdfai_extractor_client, key: imagemagick }
```

That's it! Again, without changing _any_ code, Symfony will automatically inject the correct extractor client whenever a constructor variable of type `OneToMany\PDFAI\Contract\Action\ExtractDataActionInterface` or `OneToMany\PDFAI\Contract\Action\ReadMetadataActionInterface` is present.

## Run Static Analysis
```shell
./vendor/bin/phpstan
```

## Credits
- [Vic Cherubini](https://github.com/viccherubini), [1:N Labs, LLC](https://1tomany.com)

## License
The MIT License
