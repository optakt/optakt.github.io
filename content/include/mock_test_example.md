    // ...
    t.Run("handles failure to load checkpoint", func(t *testing.T) {
    t.Parallel()
    
            tr, st := baselineFSM(t, StatusEmpty)
    
            load := mocks.BaselineLoader(t)
            load.CheckpointFunc = func() (*trie.MTrie, error) {
                return nil, mocks.GenericError
            }
    
            tr.load = load
    
            err := tr.BootstrapState(st)
            assert.Error(t, err)
        })
    // ...