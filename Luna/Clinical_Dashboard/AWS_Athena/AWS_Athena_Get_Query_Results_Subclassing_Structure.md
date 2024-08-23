# AWS Athena Get Query Results Subclassing Structure

The resulting object of:
```ruby
@athena_client.get_query_results({
  query_execution_id: @query_execution.execution_id
})
```

**It’s a very complex subclassing structure which is complicated to replicate in tests. In this doc, I show a sample result to get an idea of how to mock it.**

## General Description

- `Class Aws::Athena::Types::GetQueryResultsOutput` is the class returned by the `#get_query_results` call.
    - `#result_set` is a method that instances `Aws::Athena::Types::ResultSet`
        - `#rows` is a method that returns an array of instances of `Aws::Athena::Types::Row`
            - `#data` is a method that returns an array of instance of `Aws::Athena::Types::Datum`
                - This is where the values we need are actually stored
                - `#var_char_value` is the last thing we call to get the value

## Physicians

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = 'physician_name',
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = 'physician_group'
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = '',
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = ''
		]
	]
```


## Visits by Injury Type

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "injury_name",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "completed_visits_count"
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "Shoulder/Arm",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "43"
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "Rekha Avirah"
		]
	]
```


## Age Distribution

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "age",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "gender"
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "57",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "female"
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "William Workman"
		]
	]
```


## Case Distribution

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "injury_name",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "Knee",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "William Workman"
		]
	]
```


## Change In Pain Level

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "injury_name",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "pain_0",
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "pain_1",
		  [3] Class Aws::Athena::Types::Datum
			var_char_value = "pain_2",
		  [4] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "Knee",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "10",
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "7",
		  [3] Class Aws::Athena::Types::Datum
			var_char_value = "3",
		  [4] Class Aws::Athena::Types::Datum
			var_char_value = "William Workman"
		]
	]
```


## Change In PSFS Scale

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "injury_name",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "psfs_0",
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "psfs_1",
		  [3] Class Aws::Athena::Types::Datum
			var_char_value = "psfs_2",
		  [4] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "Knee",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "1",
		  [2] Class Aws::Athena::Types::Datum
			var_char_value = "4",
		  [3] Class Aws::Athena::Types::Datum
			var_char_value = "9",
		  [4] Class Aws::Athena::Types::Datum
			var_char_value = "William Workman"
		]
	]
```


## Quality of Life

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	  [0] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "quality_of_life",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "physician_name"
		]
	  [1] Class Aws::Athena::Types::Row
		data = [
		  [0] Class Aws::Athena::Types::Datum
			var_char_value = "good",
		  [1] Class Aws::Athena::Types::Datum
			var_char_value = "William Workman"
		]
	]
```


## Patients Treated

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
	rows = [
	 [0] Class:0x00007fb57f9752c0>:Aws::Athena::Types::Row:0x7fb5829081b8
		data = [
		  [ 0] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bf10
			  var_char_value = "first_name"
>,
		  [ 1] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58be48
			  var_char_value = "physician_name"
>,
		  [ 2] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bda8
			  var_char_value = "completed_visits_count"
>,
		  [ 3] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bd08
			  var_char_value = "pending_visits_count"
>,
		  [ 4] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bc18
			  var_char_value = "pain_0"
>,
		  [ 5] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bb78
			  var_char_value = "psfs_0"
>,
		  [ 6] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58bab0
			  var_char_value = "pain_1"
>,
		  [ 7] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58ba10
			  var_char_value = "psfs_1"
>,
		  [ 8] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b970
			  var_char_value = "pain_2"
>,
		  [ 9] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b7e0
			  var_char_value = "psfs_2"
>,
		  [10] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b718
			  var_char_value = "pain_3"
>,
		  [11] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b650
			  var_char_value = "psfs_3"
>,
		  [12] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b560
			  var_char_value = "pain_4"
>,
		  [13] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b4c0
			  var_char_value = "psfs_4"
>,
		  [14] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b3d0
			  var_char_value = "pain_5"
>,
		  [15] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b330
			  var_char_value = "psfs_5"
>,
		  [16] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b268
			  var_char_value = "pain_6"
>,
		  [17] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b1c8
			  var_char_value = "psfs_6"
>,
		  [18] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b128
			  var_char_value = "pain_7"
>,
		  [19] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58b038
			  var_char_value = "psfs_7"
>,
		  [20] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58af98
			  var_char_value = "pain_8"
>,
		  [21] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58aed0
			  var_char_value = "psfs_8"
>,
		  [22] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58ae30
			  var_char_value = "pain_9"
>,
		  [23] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58ad90
			  var_char_value = "psfs_9"
>,
		  [24] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58ac28
			  var_char_value = "discharged"
>,
		  [25] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58ab60
			  var_char_value = "injury_name"
>,
		  [26] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58aa70
			  var_char_value = "form_type"
>,
		  [27] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a9d0
			  var_char_value = "answers_0"
>,
		  [28] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a930
			  var_char_value = "answers_1"
>,
		  [29] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a7a0
			  var_char_value = "answers_2"
>,
		  [30] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a688
			  var_char_value = "answers_3"
>,
		  [31] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a5e8
			  var_char_value = "answers_4"
>,
		  [32] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a548
			  var_char_value = "answers_5"
>,
		  [33] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a430
			  var_char_value = "answers_6"
>,
		  [34] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a390
			  var_char_value = "answers_7"
>,
		  [35] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a2c8
			  var_char_value = "answers_8"
>,
		  [36] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a228
			  var_char_value = "answers_9"
>,
		  [37] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f58a160
			  var_char_value = "last_name"
>
		]
>,
	  [1] Class:0x00007fb57f9752c0>:Aws::Athena::Types::Row:0x7fb57f589df0
		data = [
		  [ 0] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589c10
			  var_char_value = "Amy"
>,
		  [ 1] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589b70
			  var_char_value = ""
>,
		  [ 2] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589aa8
			  var_char_value = "1"
>,
		  [ 3] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589a08
			  var_char_value = "1"
>,
		  [ 4] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589940
			  var_char_value = "5"
>,
		  [ 5] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589698
			  var_char_value = "8"
>,
		  [ 6] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5895f8
			  var_char_value = nil
>,
		  [ 7] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589530
			  var_char_value = nil
>,
		  [ 8] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589490
			  var_char_value = nil
>,
		  [ 9] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5893c8
			  var_char_value = nil
>,
		  [10] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5892d8
			  var_char_value = nil
>,
		  [11] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589238
			  var_char_value = nil
>,
		  [12] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589198
			  var_char_value = nil
>,
		  [13] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5890f8
			  var_char_value = nil
>,
		  [14] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f589058
			  var_char_value = nil
>,
		  [15] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588fb8
			  var_char_value = nil
>,
		  [16] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588f18
			  var_char_value = nil
>,
		  [17] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588d60
			  var_char_value = nil
>,
		  [18] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588cc0
			  var_char_value = nil
>,
		  [19] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588bf8
			  var_char_value = nil
>,
		  [20] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588b58
			  var_char_value = nil
>,
		  [21] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588a68
			  var_char_value = nil
>,
		  [22] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588928
			  var_char_value = nil
>,
		  [23] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588810
			  var_char_value = nil
>,
		  [24] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5886f8
			  var_char_value = "false"
>,
		  [25] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588658
			  var_char_value = "Hip"
>,
		  [26] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588540
			  var_char_value = "lefs"
>,
		  [27] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588360
			  var_char_value = "59"
>,
		  [28] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5881d0
			  var_char_value = nil
>,
		  [29] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f5880b8
			  var_char_value = nil
>,
		  [30] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57f588018
			  var_char_value = nil
>,
		  [31] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb23ef0
			  var_char_value = nil
>,
		  [32] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb23db0
			  var_char_value = nil
>,
		  [33] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb23cc0
			  var_char_value = nil
>,
		  [34] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb23ba8
			  var_char_value = nil
>,
		  [35] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb23ab8
			  var_char_value = nil
>,
		  [36] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb239a0
			  var_char_value = nil
>,
		  [37] Class:0x00007fb57edbe1e8>:Aws::Athena::Types::Datum:0x7fb57fb238b0
			  var_char_value = "Abeloff"
>
		]
>
```

