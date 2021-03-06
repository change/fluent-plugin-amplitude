# fluent-plugin-amplitude
Output plugin for [Fluentd](http://fluentd.org) to [Amplitude](https://amplitude.com/)

This plugin uses the [amplitude-api](https://github.com/toothrot/amplitude-api) gem, which itself talks to Amplitude's [HTTP API](https://amplitude.zendesk.com/hc/en-us/articles/204771828-HTTP-API).

## Installation
Install with gem or fluent-gem command as:

```bash
# for fluentd
$ gem install fluent-plugin-amplitude

# for td-agent
$ sudo /usr/lib64/fluent/ruby/bin/fluent-gem install fluent-plugin-amplitude
```

## Configuration

#### Example configuration
```xml
<match output.amplitude.*>
  @type amplitude
  api_key XXXXXX
  user_id_key user_id
  device_id_key uuid
  time_key created_at
  user_properties first_name, last_name
  event_properties current_source
  properties_blacklist user
  events_whitelist petition_view, petition_share
  events_blacklist email_send
  flush_interval 5s
  retry_limit 2
  remove_tag_prefix output.amplitude.
  <secondary>
    @type file
    path /var/log/fluent/forward-failed
  </secondary>
</match>
```

#### api_key
AmplitudeOutput needs your Amplitude `api_key` ([see Amplitude for more information](https://amplitude.zendesk.com/hc/en-us/articles/206728448-Where-can-I-find-my-app-s-API-Key-or-Secret-Key-))

#### user_id_key and device_id_key
You must set at least one of `user_id_key` and `device_id_key`. They will be used to pull out the `user_id` and `device_id` values from the record to send to the Amplitude API. Note these can both be arrays, and the first matching key will be used.

#### time_key
If set, `time_key` will be used to pull out a timestamp field to set as `time` in the Amplitude API request. This can be an array, the first matching key will be used.

####  revenue_properties

`revenue_properties` are a fixed set of properties that Amplitude looks for in order to determine which events are revenue events (see Amplitude's [revenue tracking guide](https://amplitude.zendesk.com/hc/en-us/articles/115003116888-Tracking-Revenue) and [HTTP API documentation](https://amplitude.zendesk.com/hc/en-us/articles/204771828#keys-for-the-event-argument) for more information). `revenue_properties` are set as top-level properties on the API call. The `revenue_properties` we currently support are:
* quantity
* price
* revenue
* revenue_type

#### user_properties and event_properties
You can optionally specify lists of `user_properties` and `event_properties` to pull from the record.

If `user_properties` are specified, only those properties will be included as `user_properties` in the Amplitude API call.  Otherwise no `user_properties` will be sent.

If `event_properties` are specified, only those properties will be included as `event_properties` in the Amplitude API call. If no `event_properties` are specified, then every field with the exception of `user_id_key`, `device_id_key`, all `revenue_properties`, and anything specified in `user_properties`, will be sent as `event_properties` in the Amplitude API call.

#### event type
The event_type is the tag.  To modify this, fluent-plugin-amplitude includes the `HandleTagNameMixin` mixin which allows the following options:

```xml
remove_tag_prefix <tag_prefix_to_remove_including_the_dot>
remove_tag_suffix <tag_suffix_to_remove_including_the_dot>
add_tag_prefix <tag_prefix_to_add_including_the_dot>
add_tag_suffix <tag_suffix_to_add_including_the_dot>
```

#### properties_blacklist
Any properties included in the blacklist will be scrubbed from the record.

#### events_whitelist
If your `<match>` is using a wildcard, you can specify specific events to whitelist. If the `events_whitelist` is empty all events will be sent to Amplitude. Note the event name here is the `event_type` (so should not include, e.g., any prefixes that were removed via `remove_tag_prefix`)

#### events_blacklist
If your `<match>` is using a wildcard, you can specify specific events to blacklist. If the `events_blacklist` is empty all events will be sent to Amplitude. Note the event name here is the `event_type` (so should not include, e.g., any prefixes that were removed via `remove_tag_prefix`). It probably only makes sense to use `events_whitelist` or `events_blacklist`.

#### Error handling
Any error will result in the message retrying. In the case of an incorrectly configured API key, this can result in the messages infinitely retrying.  You should set the normal [buffered output plugin options](http://docs.fluentd.org/articles/buffer-plugin-overview) to prevent this (and to preserve data in the case of misconfigured records failing to be submitted to Amplitude).

## Contributing

1. Fork it ( http://github.com/change/fluent-plugin-amplitude/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
