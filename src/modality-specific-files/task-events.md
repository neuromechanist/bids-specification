# Task events

The purpose of this file is to describe timing and other properties of events
recorded during a run.
Events are, for example, stimuli presented to the participant or participant responses
(see [Definitions](../common-principles.md#definitions)).
A single event file MAY include any combination of stimulus, response, and other events.
Events MAY overlap in time.
Please mind that this does not imply that only so called "event related" study designs
are supported (in contrast to "block" designs) - each "block of events" can be
represented by an individual row in the `events.tsv` file (with a long
duration).

Template:

```Text
sub-<label>/[ses-<label>]
    <data_type>/
        <matches>_events.tsv
        <matches>_events.json
```

Where `<matches>` corresponds to task filename. For example:
`sub-control01_task-nback`.

Each task events file REQUIRES a corresponding task data file.
It is also possible to have a single `events.tsv` file describing events
for all participants and runs (see
[Inheritance Principle](../common-principles.md#the-inheritance-principle)).
As with all other tabular data, `events.tsv` files MAY be accompanied
by a JSON file describing the columns in detail
(see [Tabular Files](../common-principles.md#tabular-files)).

The tabular files consists of one row per event and a set of REQUIRED
and OPTIONAL columns:

<!-- This block generates a columns table.
The definitions of these fields can be found in
  src/schema/rules/tabular_data/*.yaml
and a guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_columns_table("task.TaskEvents") }}

The content of  `events.tsv` files SHOULD be sorted by values in the `onset` column.

An arbitrary number of additional columns can be added.
Those allow describing other properties of events that could be later referenced in modeling and
hypothesis extensions of BIDS.
Note that the `trial_type` and any additional columns in a TSV file
SHOULD be documented in an accompanying JSON sidecar file.

!!! note "Regarding the precision of numeric metadata"

    For the precision of numeric metadata,
    it is RECOMMENDENDED that dataset curators specify numeric metadata like `onset` and
    `duration` with as much decimal precision as is reasonable in the context of the experiment.
    For example in an EEG experiment with devices operating at 1000 Hz sampling frequency,
    dataset curators SHOULD specify **at least** 3 figures after the decimal point.

!!! note "For fMRI data"

    For fMRI data
    if any acquired scans have been discarded before forming the imaging data file,
    ensure that an `onset` of 0 corresponds to the time the first image was stored.
    For example in case there is an in scanner training phase that
    begins before the scanning sequence has started events from this sequence should
    have negative onset time counting down to the beginning of the acquisition of
    the first volume.

Example:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-control01": {
      "func": {
         "sub-control01_task-stopsignal_events.tsv": "",
         "sub-control01_task-stopsignal_events.json": "",
         },
      },
   }
) }}

Example of the content of the TSV file:

```Text
onset	duration	trial_type	response_time	stim_file	channel	annots
1.23	0.65	start	1.435	images/red_square.jpg	n/a	n/a
5.65	0.65	stop	1.739	images/blue_square.jpg	n/a	n/a
12.1	2.35	n/a	n/a	n/a	F,1|F,2|Cz	musc
```

In the accompanying JSON sidecar, the `trial_type` column might look as follows:

```JSON
{
    "trial_type": {
        "LongName": "Event category",
        "Description": "Indicator of type of action that is expected",
        "Levels": {
            "start": "A red square is displayed to indicate starting",
            "stop": "A blue square is displayed to indicate stopping"
        }
    },
    "channel": {
        "Description": "Channel(s) associated with the event",
        "Delimiter": "|"
    },
    "annots": {
        "LongName": "Annotations",
        "Description": "Annotations associated with channels indicated in the channel column.",
        "Levels": {
            "musc": "Muscle artifact. A very common, high frequency, sharp artifact that corresponds with agitation/nervousness in a patient."
        },
        "HED": {
            "musc": "EMG-artifact"
        }
    }
}
```

!!! note

    In the example above:

    1.  Only a subset of columns are described for the sake of brevity.
        In a real dataset, all other columns SHOULD also be described.

    1.  The `channel` column contains a list of values that are separated
        by a delimiter (`|`), as is declared in the `Delimiter` metadata field
        of the `events.json` file.
        Thus, the channels related to the event in the third row of the example
        are called `F,1`, `F,2`, and `Cz`.

    1.  The example contains a column called `annots`.
        This column is not defined in BIDS, and constitutes additional,
        arbitrary (that is, unofficial) metadata.
        In the present case, it is used to describe artifacts in the data,
        in reference to the `channel` column.
        The `annots` column is making use of the powerful HED system
        for documenting events, see below.

Events MAY also be documented in machine-actionable form
using HED (Hierarchical Event Descriptor) tags.
This type of documentation is particularly useful for datasets likely to be used
in event-related analyses.
See [Hierarchical Event Descriptors](../appendices/hed.md)
for additional information and examples.

For multi-echo files, the `events.tsv` file is applicable
to all echos of a particular run:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-01": {
      "func": {
        "sub-01_task-cuedSGT_run-1_events.tsv": "",
        "sub-01_task-cuedSGT_run-1_echo-1_bold.nii.gz": "",
        "sub-01_task-cuedSGT_run-1_echo-2_bold.nii.gz": "",
        "sub-01_task-cuedSGT_run-1_echo-3_bold.nii.gz": "",
         },
      },
   }
) }}

## Stimuli

Additional information about the stimuli can be added in the `events.tsv`
and `events.json` files.

This can be done by using a `/stimuli` directory or by reference to a stimuli database.

### Stimuli directory

The stimulus files can be added in a `/stimuli` directory
(under the root directory of the dataset; with optional subdirectories) AND using a
`stim_file` column in `events.tsv` mentioning which stimulus file was used
for a given event.

It is RECOMMENDED to include a `stimuli.tsv` table containing a list of the stimuli present in the `/stimuli` directory.
The `stimuli.tsv` would list the stimulus files as rows and provide more information about the stimulus files as columns, such as their types (for example, still images, movies, audio, multi-channel audio, etc), text descriptions, and optional columns including references, relevant HED tags, etc.
Annotations for the still stimulus files can be properly held in `stimuli.tsv` as specific columns. The `stimuli.tsv` file can be accompanied by a `stimuli.json` file describing the columns in detail. The `stimuli.json`, similar to the `events.json` file, MAY also include additional information about the stimuli, such as references to databases, HED tags, etc.
The `stimuli.tsv` MUST include the following columns:

{{ MACROS___make_columns_table(
   {
      "stim_file": ("REQUIRED", "There are no restrictions on the file formats of the stimuli files, but they should be stored in the `/stimuli` directory (under the root directory of the dataset; with OPTIONAL subdirectories). The values under the stim_file column correspond to a path relative to `/stimuli`. For example `images/cat03.jpg` will be translated to `/stimuli/images/cat03.jpg`. This column MUST appear as the first column in the file."),
      "type": ("REQUIRED", "String. One of `movie`, `still_image`, `audio`, `annotation`, or Freeform.")
   }
) }}

### Time-varying stimuli
A single-line annotation in a `stimuli.tsv` column would be sufficient for still stimulus files (for example, images). However, a time-varying stimulus file could need separate *temporal annotations* (or traces) for every frame (that is, the smallest temporal resolution of the stimulus file).

The temporal annotations for time-varying stimuli can be stored in a separate file, `stimulus-file-name_trace-XXX_annot.tsv`, which would hold the annotations for time points of the time-varying stimulus similar to an `events.tsv` file. The temporal annotation filename MUST start with the stimulus file name available in the `/stimuli` directory. Difference annotation traces can be differentiated by modifying the trace name. The `annot.tsv` MUST follow the same format and contain the same REQUIRED and OPTIONAL columns of the `events.tsv` file, and MAY be accompanied by a JSON file describing the columns in detail.

An exemplary `stimuli.tsv` file is shown below:

| stim_file                            | type        | NSD_id | COCO_id | first_COCO_description                         | HED                                                                                                                                                                                            |
| ------------------------------------ | ----------- | ------ | ------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| nsd03050.png                         | still_image | 3050   | 262414  | a person standing on a surfboard riding a wave | (Foreground-view, ((Item-count/1, ((Human, Human-agent), Body, Male, Agent-trait/Adolescent)), (Play, (Item-count/1, Man-made-object)))), (Background-view, (Outdoors, Natural-feature/Ocean)) |
| the-present_movie.mp4                | movie       | n/a    | n/a     | n/a                                            | Visual-presentation, Movie                                                                                                                                                                     |
the-present_trace-LogLumRatio_annot.tsv | annotation  | n/a    | n/a     | n/a                                            | n/a                                                                                                                                                                                            |

An exemplary `events.tsv` file is shown below:

| onset  | duration | value    | stim_file                         | HED                                                         |
| ------ | -------- | -------- | --------------------------------- | ----------------------------------------------------------- |
| 25.033 | 1.02     | nsd03050 | nsd03050.png | Sensory-event, Visual-presentation, (Image, {nsd03050.png}) |

In this example, the `nsd03050.jpg` points to the respective row in `stimuli.tsv` file. The analyst can then expand the `events.tsv` using the associated columns populated for this row. Also, following the HED-specifications 3.2.0, the HED annotation `{nsd03050.jpg}` indicates inserting the HED annotation from the `HED` column in the `stimuli.tsv` file to the `events.tsv` file's `HED` column for that particular event row.

### Relation to  `events.tsv` and `stim.tsv.gz` files
The columns and annotations of an `events.tsv` file can be exapnded by annotations within the `stimuli.tsv` and `stimulus.tsv` files using remodelers when the file name is mentioned in the `stim_file` or other columns.

While the contents of `stimuli.tsv` and `stimulus.tsv` can also be included in the individual `events.tsv` files, such implementation unnecessarily replicates annotations across subjects, modalities, tasks, and runs. Also, using `stimuli.tsv` and `stimulus.tsv` increases the possibility of both (1) reusing the same annotations in other studies and (2) reusing the dataset with alternative annotations - by merely changing one file.

Another advantage of top-level `stimuli.tsv` and `stimulus.tsv` is avoiding the need to create and edit large `events.tsv` (and `events.json`) files for datasets that use complex stimuli, such as movies. By avoiding the need to include annotations in every data folder, the remodeler software will retrieve these annotations housed in the single top-level `/stimuli` directory during the analysis. This increases the readability of data-level `events.tsv` and `events.json` files and avoid using fixed, lengthy annotations to annotate every instance of a constant stimulus. For large, standard stimulus sets, such as the [Natural Scene Dataset](https://naturalscenesdataset.org) or the immense [COCO image dataset](https://cocodataset.org/), holding a list of images used across the BIDS dataset in the top-level `/stimuli` directory, with their annotations, would give a useful cross-reference for debugging and design of analyses.

### Stimuli databases

References to existing databases can also be encoded using additional columns.
The following example includes references to the
[Karolinska Directed Emotional Faces (KDEF) database](https://www.kdef.se/).

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example(
   {
   "sub-control01": {
      "func": {
         "sub-control01_task-emoface_events.tsv": "",
         "sub-control01_task-emoface_events.json": "",
         },
      },
   }
) }}

Example of the content of the TSV file:

```Text
onset duration  trial_type  identifier  database  response_time
1.2 0.6 afraid  AF01AFAF  kdef  1.435
5.6 0.6 angry AM01AFAN  kdef  1.739
5.6 0.6 sad AF01ANSA  kdef  1.739
```

The `trial_type` and `identifier` columns from the `events.tsv` files might be described
in the accompanying JSON sidecar as follows:

```JSON
{
    "trial_type": {
        "LongName": "Emotion image type",
        "Description": "Type of emotional face from Karolinska database that is displayed",
        "Levels": {
            "afraid": "A face showing fear is displayed",
            "angry": "A face showing anger is displayed",
            "sad": "A face showing sadness is displayed"
        }
    },
    "identifier": {
        "LongName": "Karolinska (KDEF) database identifier",
        "Description": "ID from KDEF database used to identify the displayed image"
    }
}
```

Note that all other columns SHOULD also be described but are omitted
for the sake of brevity.

### Stimulus presentation details

It is RECOMMENDED to include details of the stimulus presentation software,
when applicable:

<!-- This block generates a metadata table.
These tables are defined in
  src/schema/rules/sidecars
The definitions of the fields specified in these tables may be found in
  src/schema/objects/metadata.yaml
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_sidecar_table("events.StimulusPresentation") }}

The object supplied for `StimulusPresentation` SHOULD include the following key-value pairs:

<!-- This block generates a table describing subfields within a metadata field.
The definitions of these fields can be found in
  src/schema/objects/metadata.yaml
and a guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_subobject_table("metadata.StimulusPresentation") }}

The operating system description SHOULD include the following attributes:

-   type (for example, Windows, macOS, Linux)
-   distribution (if applicable, for example, Ubuntu, Debian, CentOS)
-   the version number (for example, 18.04.5)

Examples:

-   Windows 10, Version 2004
-   macOS 10.15.6
-   Linux Ubuntu 18.04.5

The amount of information supplied for the `OperatingSystem` SHOULD be sufficient
to re-run the code under maximally similar conditions.

The information related to stimulus presentation might be described
in the accompanying JSON sidecar as follows (based on the example of the previous section):

```JSON
{
    "trial_type": {
        "LongName":   "Emotion image type",
        "Description": "Type of emotional face from Karolinska database that is displayed",
        "Levels": {
            "afraid": "A face showing fear is displayed",
            "angry":  "A face showing anger is displayed",
            "sad":    "A face showing sadness is displayed"
        }
    },
    "identifier": {
        "LongName": "Unique identifier from Karolinska (KDEF) database",
        "Description": "ID from KDEF database used to identify the displayed image"
    },
    "StimulusPresentation": {
        "OperatingSystem": "Linux Ubuntu 18.04.5",
        "SoftwareName": "Psychtoolbox",
        "SoftwareRRID": "SCR_002881",
        "SoftwareVersion": "3.0.14",
        "Code": "doi:10.5281/zenodo.3361717",
        "ScreenDistance": 1.8,
        "ScreenRefreshRate": 60,
        "ScreenResolution": [1920, 1200],
        "ScreenSize": [0.472, 0.295],
        "HeadStabilization": "none"
    },
    "VisionCorrection": "lenses"
}
```

### Continuously-sampled, stimulus-related signals

!!! example "Example datasets"

      The following [BIDS-Examples](https://bids-standard.github.io/bids-examples/#dataset-index)
      showcase stimulus-related signals and may be used as a reference
      when curating a new dataset:

      -   ["synthetic" example dataset](https://github.com/bids-standard/bids-examples/tree/master/synthetic).

Signals related to stimuli (such as parameters of a film or audio stimuli) that are
evenly recorded at a constant sampling frequency MUST be specified using a
[compressed tabular file](../common-principles.md#compressed-tabular-files)
([TSV.GZ file](../glossary.md#tsv_gz-extensions)) and a corresponding
JSON file for storing metadata fields (see below).

Template:

```Text
sub-<label>/[ses-<label>/]
    <datatype>/
        <matches>_stim.tsv.gz
        <matches>_stim.json
```

For the template directory name, `<datatype>` can correspond to any data
recording modality.

In the template filenames, the `<matches>` part corresponds to task filename
before the suffix.
For example for the file `sub-control01_task-nback_run-1_bold.nii.gz`,
`<matches>` would correspond to `sub-control01_task-nback_run-1`.

!!! warning "Caution"

    `<matches>_stim.tsv.gz` files MUST NOT include a header line,
    as established by the [common-principles](../common-principles.md#compressed-tabular-files).
    As a result, when supplying a `<matches>_stim.tsv.gz` file, an accompanying
    `<matches>_stim.json` MUST be present to indicate the column names.

If the same continuous recording has been used for all subjects (for example in
the case where they all watched the same movie), one file placed in the
root directory (for example, `<root>/task-movie_stim.<tsv.gz|json>`) MAY be used
and will apply to all `<matches>_task-movie_<matches>_<suffix>.<ext>` files.
In the following example, the two `task-nback_stim.<json|tsv.gz>` apply
to all the `task-nback` runs across the two available subjects:

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_filetree_example({
  "sub-01": {
    "func": {
      "sub-01_task-nback_run-1_bold.nii.gz": "",
      "sub-01_task-nback_run-2_bold.nii.gz": "",
    },
  },
  "sub-02": {
    "func": {
      "sub-02_task-nback_run-1_bold.nii.gz": "",
      "sub-02_task-nback_run-2_bold.nii.gz": "",
    },
  },
  "task-nback_stim.json": "",
  "task-nback_stim.tsv.gz": "",
}) }}

The following table specifies metadata fields for the
`<matches>_stim.json` file.

<!-- This block generates a metadata table.
These tables are defined in
  src/schema/rules/sidecars
The definitions of the fields specified in these tables may be found in
  src/schema/objects/metadata.yaml
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->
{{ MACROS___make_sidecar_table(["continuous.Continuous"]) }}

Additional metadata may be included as in
[any TSV file](../common-principles.md#tabular-files) to specify, for
example, the units of the recorded time series for each column.
