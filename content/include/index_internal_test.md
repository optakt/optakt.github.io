
    func TestIndex(t *testing.T) {
        // ...
        t.Run("collections", func(t *testing.T) {
            t.Parallel()
    
            collections := mocks.GenericCollections(4)
    
            reader, writer, db := setupIndex(t)
            defer db.Close()
    
            assert.NoError(t, writer.Collections(mocks.GenericHeight, collections))
            // Close the writer to make it commit its transactions.
            require.NoError(t, writer.Close())
    
            // NOTE: The following subtests should NOT be run in parallel, because of the deferral
            // to close the database above.
            t.Run("retrieve collection by ID", func(t *testing.T) {
                got, err := reader.Collection(collections[0].ID())
    
                require.NoError(t, err)
                assert.Equal(t, collections[0], got)
            })
    
            t.Run("retrieve collections by height", func(t *testing.T) {
                got, err := reader.CollectionsByHeight(mocks.GenericHeight)
    
                require.NoError(t, err)
                assert.ElementsMatch(t, mocks.GenericCollectionIDs(4), got)
            })
    
            t.Run("retrieve transactions from collection", func(t *testing.T) {
                // For now this index is not used.
            })
        })
        // ...
    }