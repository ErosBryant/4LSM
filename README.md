# 4LSM Rocksdb 

### Get key - memtable 

```
Status DBImpl::GetImpl(const ReadOptions& read_options, const Slice& key,
                       GetImplOptions& get_impl_options) 

...
    if (get_impl_options.get_value) {
         // **get key from mem -zhao**
        if (sv->mem->Get(
              lkey,
              get_impl_options.value ? get_impl_options.value->GetSelf()
                                     : nullptr,
              get_impl_options.columns, timestamp, &s, &merge_context,
              &max_covering_tombstone_seq, read_options,
              false /* immutable_memtable */, get_impl_options.callback,
              get_impl_options.is_blob_index)) {
        done = true;

        if (get_impl_options.value) {
          get_impl_options.value->PinSelf();
        }

        RecordTick(stats_, MEMTABLE_HIT);
        
      } else if ((s.ok() || s.IsMergeInProgress()) &&
       // **get key from IMM mem -zhao**
                 sv->imm->Get(lkey,
                              get_impl_options.value
                                  ? get_impl_options.value->GetSelf()
                                  : nullptr,
                              get_impl_options.columns, timestamp, &s,
                              &merge_context, &max_covering_tombstone_seq,
                              read_options, get_impl_options.callback,
                              get_impl_options.is_blob_index)) {
        done = true;

        if (get_impl_options.value) {
          get_impl_options.value->PinSelf();
        }

        RecordTick(stats_, MEMTABLE_HIT);
        
      }
```

### Get key - SSTable 


```

void Version::Get(const ReadOptions& read_options, const LookupKey& k,
                  PinnableSlice* value, PinnableWideColumns* columns,
                  std::string* timestamp, Status* status,
                  MergeContext* merge_context,
                  SequenceNumber* max_covering_tombstone_seq,
                  PinnedIteratorsManager* pinned_iters_mgr, bool* value_found,
                  bool* key_exists, SequenceNumber* seq, ReadCallback* callback,
                  bool* is_blob, bool do_merge){

                  ....

 //  search sstale -- zhao
    *status = table_cache_->Get(
        read_options, *internal_comparator(), *f->file_metadata, ikey,
        &get_context, mutable_cf_options_.block_protection_bytes_per_key,
        mutable_cf_options_.prefix_extractor,
        cfd_->internal_stats()->GetFileReadHist(fp.GetHitFileLevel()),
        IsFilterSkipped(static_cast<int>(fp.GetHitFileLevel()),
                        fp.IsHitFileLastInLevel()),
        fp.GetHitFileLevel(), max_file_size_for_l0_meta_pin_);
        ....

              case GetContext::kFound:
        // get key from file  sstable -zhao 
        if (fp.GetHitFileLevel() == 0) {
          RecordTick(db_statistics_, GET_HIT_L0);
          trace_zhao::get_log<< "level 0" << "\n";
        } else if (fp.GetHitFileLevel() == 1) {
          RecordTick(db_statistics_, GET_HIT_L1);
          trace_zhao::get_log<< "level 1" << "\n";
        } else if (fp.GetHitFileLevel() >= 2) {
          RecordTick(db_statistics_, GET_HIT_L2_AND_UP);
          trace_zhao::get_log<< "level 2 and mor" << "\n";
        }

```