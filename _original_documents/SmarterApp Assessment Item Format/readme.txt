The zip archive contains the complete set of files for the
SmarterApp Assessment Item Format Specification
V 1.0
as of 2014-09-30.

Directory: SAAIF Spec Doc:
   readme.txt -- This file.

   The specification document and its parts:

   SmarterApp Assessment Item Format Specification 20140930 Public Release.docx
   -- The V 1.0 Specification Word Document.
   SmarterApp Assessment Item Format Specification 20140930 Public Release.pdf
   -- The V 1.0 Specification as PDF.

   Subdirectory: Document Equation Editor Tabs:
      14 files.
      XML for the standard SBAC Equation Editor Configuration Documents.
      One file for each of the standard tab configurations (11 files)
         e.g., SBAC3, SBAC4, ...
      The partial contents of the file are used within the Tab listings in the Specification.
      Additional files describe how to validate the tab descriptions.

   Subdirectory: Document Examples:
      8 files.
      The XML for the items that are in the Specification.
      One file for each of the examples (7 files) in the Specification.
      An additional file describing how to validate an example.

   Subdirectory: Document Illustrations:
      21 files.
      A single Visio file with the source of the illustrations in the Specification.
      A .png for each illustration (at 600x600 dpi).
      The .png file are used in the Specification.

   Subdirectory: Document XSD Diagrams:
      49 files.
      The XSD schema diagrams that are in the Specification.
   If appropriate, assign a new version number (the XSD versioning strategy is described in the Specification).
   If the change involves the assessmentitemtypes_v1p0.xsd or saaifitemtypes_v1p0.xsd, make changes to both the strict and lax versions of the file.
   Update the index.html file as appropriate.
   Update the Specification as appropriate.

Changing the Specification: 
   Make any editorial and technical changes.
   If a figure or table is added, the entry must be added to the list of figures/table and all subsequent figures/tables need to be renumbered (in the list, in the caption and wherever referenced in the Specification).
   If an element or attribute is added, include the indexing tags.
      Short names require special attention to indexing to get leader dots.
   If an XSD changes:
      Generate new diagrams to show the changed XSD.
      Validate any examples impacted by the change:
         Update the date of validation on the example in the Specification.
      Update any location/version changes in the tables listing the XSDs.
   If an example changes or the change impacts an example:
      Update the example.
      Validate the change.
         Update the date of validation on the example in the Specification.
   Add appropriate notes in the change log in the Specification and assign a new Specification version number (the document versioning strategy is described in the specification).
   Update the version number of the title page and the page footers.
   Update the date on the title page and page footers.
   Review the location of all page breaks in the entire document and adjust as needed.
   Verify all page headers are correct.
   Generate a new index:
      Manually add the blank lines between sections.
   Update the page numbers of list of table, figures and code listing.
   Update the Content:
      If new entries are not added, only update the page numbers.
      Manually add the blank lines between sections.
   Add/delete a blank page after the Code listing so Page 1 is an odd facing page in duplex (note, this may break page headers, footers here).
   Generate the PDF (high resolution).
   Assign an appropriate title and other properties to the PDF.


Known Issues:
   The Specification and XSDs require the accessibility elements be non empty.
   Some AIR items include empty accessibility elements.
   If non empty elements are allowed, the Specification and XSDs will need to be changed (Specification changes will include new diagrams).

   The Specification and XSDs do not support annotation elements
   Some AIR items include annotation elements.
   If annotation elements are allowed, the Specification and XSDs will need to be changed (Specification changes will include new diagrams).

   The Specification and XSDs require integer minval/maxval attributes on samplelist elements.
   Some AIR items include non integer attributes.
   If the attribute definition changes, the Specification and XSDs will need to be changed (Specification changes will include new diagrams).

   The XSDs support either strict (namespaced) or lax (any string) validation of xHTML elements.
   AIR items wrap xHTML is CDATA, so lax validation is required.
   Strict validation would require that the xHTML not be CDATA wrapped
   Strict validation would require that the xHTML elements be namespaced.

   AIR items wrap QTI in CDATA.
   Strict validation would require that the QTI not be CDATA wrapped.
   Current lax validation accepts any string.
   Current strict validation requires that the QTI be namespaced (any namespace).
   Formal QTI validation is not performed.
   AIR may be using non standard QTI.

   AIR items wrap GRID in CDATA.

   AIR items have recursive question elements within the preset answer part of a GRID item.
   Recursive definitions are not supported in XSD and validation.
   Extraneous elements need to be eliminated for a grid item to validate.   

