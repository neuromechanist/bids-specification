# Stimuli

## Stimulus Files Organization

Stimulus files MAY be stored in the `/stimuli` directory under the root directory of the dataset.
When this specification is followed,
stimulus files in the `/stimuli` directory MUST follow the naming conventions described below,
and `events.tsv` files reference them through the `stim_id` column.

The standardization of stimulus files and their annotations within BIDS offers several key benefits:

1.  **Consistency**: Ensures uniform storage and referencing across datasets
1.  **Reusability**: Enables stimulus reuse across studies through standardized structure
1.  **Efficiency**: Minimizes redundancy by centralizing annotations
1.  **Flexibility**: Facilitates dataset reuse with alternative annotations

To preserve backward compatibility with existing datasets
(which reference stimulus files directly through the `stim_file` column,
as described in [Task events](events.md#stimulus-organization)),
following this specification for the `/stimuli` directory
and using the `stim_id` column in `events.tsv` files is RECOMMENDED but not required.
Researchers are encouraged to follow these guidelines to enhance the interoperability and reproducibility of their studies.

## File Organization

Stimulus files, their metadata, and their annotations are stored directly in the `/stimuli` directory:

```Text
stimuli/
    stimuli.tsv
    stimuli.json
    annotations.tsv
    annotations.json
    stim-<label>[_part-<label>]_<suffix>.<extension>
    stim-<label>[_part-<label>]_<suffix>.json
    stim-<label>[_part-<label>]_annot-<label>_events.tsv
    stim-<label>[_part-<label>]_annot-<label>_events.json
    stim-<label>_annotations.tsv
    stim-<label>_annotations.json
```

The presence of a `stimuli.tsv` file in the `/stimuli` directory indicates
that the content of the `/stimuli` directory follows this specification.

The `stim-<label>` entity identifies a stimulus:
all files that share the same `stim-<label>`
(the stimulus files, their JSON metadata, and their annotation files)
describe the same stimulus.
A stimulus MAY be stored in several files with different suffixes or extensions,
for example the separate audio and video tracks of a movie.
The `part-<label>` entity MAY be used to split a single stimulus into parts,
such as the chapters of an audiobook or the segments of a long movie.

### Stimulus File Formats

The following table lists the supported stimulus file formats and their corresponding suffixes. The suffixes are used to identify the type of stimulus file and are appended to the `stim-<label>` prefix in the file name.

| suffix      | extensions                                                | description                  |
| ----------- | --------------------------------------------------------- | ---------------------------- |
| audio       | `.wav`, `.flac`, `.mp3`, `.aac`, `.ogg`                   | Audio-only stimulus files    |
| image       | `.jpg`, `.jpeg`, `.png`, `.svg`, `.webp`, `.tif`, `.tiff` | Static visual stimulus files |
| video       | `.mp4`, `.avi`, `.mkv`, `.webm`                           | Video-only stimulus files    |
| audiovideo  | `.mp4`, `.avi`, `.mkv`, `.webm`                           | Combined audio-visual files  |

See the [Media Files appendix](../appendices/media-files.md) for format details and
the recommended technical metadata for media streams.

## Stimulus description (`stim-<label>_<suffix>.json`)

The `stim-<label>_<suffix>.json` file provides metadata about the _singular_ stimulus file.
The following fields are defined to describe the stimulus file:

<!-- This block generates a metadata table.
These tables are defined in
  src/schema/rules/sidecars
The definitions of the fields specified in these tables may be found in
  src/schema/objects/metadata.yaml
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_sidecar_table("stimuli.Stimuli") }}

In some cases, such as observing the copyright of a stimulus file, the actual stimulus file may not be shared.
In such cases, the `stim-<label>_<suffix>.json` file SHOULD be used to provide metadata about the stimulus file,
including the license, copyright, URL, and description.

### Example `stim-<label>_<suffix>.json`

```JSON
{
    "License": "CC-BY-4.0",
    "Copyright": "2023 Lab Name lab@university.edu",
    "URL": "https://example.com/stimuli/face01.png",
    "Description": "A female face with a neutral expression"
}
```

The `License` field SHOULD provide a known identifier,
such as `PDL`, `CC0`, or `CC-BY-4.0` from the [Licenses appendix](../appendices/licenses.md),
or from common license lists such as [SPDX](https://spdx.org/licenses/)
or [Creative Commons](https://creativecommons.org/licenses/).
The `Copyright` field SHOULD provide the year, copyright holder's name,
and if available, the email address of the copyright holder.
If the stimulus file is not shared, the `URL` field SHOULD provide a link to the stimulus file itself,
not to a directory or collection of files.

## Stimuli Description (`stimuli.tsv`)

The `stimuli.tsv` file is used to provide information about the stimuli based on their `stimulus_id`.
This file is similar in usage to `participants.tsv`, `scans.tsv`, and `sessions.tsv`,
which list descriptions about subjects, scans, and sessions, respectively.
A `stimuli.tsv` file in the `/stimuli` directory is RECOMMENDED.

The `stimuli.tsv` file contains information about each stimulus, including stimulus ID, type, URL, and other relevant details. The following table describes the REQUIRED, RECOMMENDED, and OPTIONAL columns for the `stimuli.tsv` file:

<!-- This block generates a columns table.
The definitions of these fields can be found in
  src/schema/rules/tabular_data/*.yaml
and a guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_columns_table("stimuli.Stimuli") }}

### Example `stimuli.tsv`

```tsv
stimulus_id	type	URL	license	copyright	description	present
stim-face01	image	https://example.com/faces/face01.jpg	CC-BY-4.0	Lab 2023	A female face with neutral expression	true
stim-tone01	audio	https://example.com/tones/tone01.wav	CC-BY-4.0	Lab 2023	A 440Hz pure tone	true
stim-movie01	video	https://example.com/movies/movie01.mp4	n/a	Studio XYZ	A clip from a copyrighted movie	false
```

The `stimuli.json` file provides detailed descriptions of the columns in the `stimuli.tsv` file. There MAY be extra entries in the `stimuli.json` in addition to the columns in the `stimuli.tsv` to provide more details about the stimulus.

In cases where the stimulus is not shared, the `stimuli.tsv` file can be used to provide metadata about the stimuli, including the license, copyright, URL, and description. This is similar to the use of `stim-<label>_<suffix>.json` files for individual stimuli files. In the case of conflict between the metadata in the `stimuli.tsv` and `stim-<label>_<suffix>.json` files, the metadata in the `stim-<label>_<suffix>.json` file takes precedence.

## Stimulus Annotations

Annotations of the still images or general description of the stimuli (such as frequency and duration of a beep sound) can be stored in the `stimuli.tsv` as an additional column or `stim-<label>_<suffix>.json` as described above. Here is an example of how annotations can be stored in the `stimuli.tsv` file for an image from the Natural Scenes Dataset (NSD):

```tsv
stimulus_id	type	description	HED	NSD_id	COCO_id
stim-nsd02951	image	an open market full of people and piles of vegetables	((Item-count, High), Ingestible-object), (Background-view, ((Human, Body, Agent-trait/Adult), Outdoors, Furnishing, Natural-feature/Sky, Urban, Man-made-object))	2951	262145
```

For time-varying stimuli, such as audio or video,
it is RECOMMENDED to store annotations in `stim-<label>_annot-<label>_events.tsv` files.
These files follow the format of `events.tsv` files,
including the REQUIRED `onset` and `duration` columns,
with onsets expressed relative to the start of the stimulus.
There can be multiple annotation files for a single stimulus,
each with a unique `annot-<label>`.
Annotation files MUST be stored in the `/stimuli` directory.
When more than one stimulus file shares the same `stim-<label>`,
the `Sources` field of the annotation file's JSON sidecar
SHOULD list the stimulus files that the annotations describe,
using [BIDS URIs](../common-principles.md#bids-uri).

## Annotation Description (`annotations.tsv`)

The `annotations.tsv` file contains additional metadata about stimulus annotations.
There MAY be a single `annotations.tsv` file for all the stimuli
or separate `stim-<label>_annotations.tsv` files for each stimulus.
The values of the `annot_id` column are the labels of the `annot-<label>` entity.
The following columns are defined for the `annotations.tsv` file:

<!-- This block generates a columns table.
The definitions of these fields can be found in
  src/schema/rules/tabular_data/*.yaml
and a guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_columns_table("stimuli.Annotations") }}

### Example `*_annotations.tsv`

```tsv
annot_id	description
emotion	Emotion expressed by the face, one row per image
gender	Perceived gender of the face, one row per image
pitch	Fundamental frequency of the tone over time
```

## Referencing Stimulus Identifiers in `events.tsv`

To reference stimulus identifiers in the `events.tsv` file, use the `stim_id` column.
Each value of the `stim_id` column MUST take the form `stim-<label>`
and refers to all files (both stimulus and annotation files) that share the same `stim-<label>`.
Each `stim_id` value SHOULD match the `stimulus_id` of an entry in `stimuli.tsv`
or the `stim-<label>` of stimulus files in `/stimuli`.
See [Task events](events.md#stimulus-organization) for using `stim_id` together with the `stim_file` column.

Example `events.tsv` file:

```tsv
onset	duration	trial_type	response_time	stim_id
1.23	0.65	face	1.435	stim-face01
5.65	0.65	face	1.739	stim-face02
12.1	2.35	tone	n/a	stim-tone01
```

In the accompanying JSON sidecar, the `stim_id` column might be described as follows:

```JSON
{
    "stim_id": {
        "LongName": "Stimulus identifier",
        "Description": "Represents a unique identifier for the stimulus presented at the given onset time."
    }
}
```
