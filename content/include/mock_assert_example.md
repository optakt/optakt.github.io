    // ...
    t.Run("nominal case", func(t *testing.T) {
        t.Parallel()

        mock := mocks.BaselineTransport(t)
        mock.Request = func(method, url string) (*http.Response, error) {
            assert.Equal(t, http.MethodPost, method)
            assert.Equal(t, "https://google.com", url)

            return testResponse, nil
        }

        subject := NewServer()

        got, err := subject.Call()
        require.NoError(t, err)
        assert.NotNil(t, got)
        // Assertions...
    })
    t.Run("handles failure to load checkpoint", func(t *testing.T) {
        t.Parallel()

        mock := mocks.BaselineTransport(t)
        mock.Request = func(string, string) (*http.Response, error) {
            return nil, mocks.GenericError()
        }

        subject := NewServer()

        _, err := tr.BootstrapState(st)
        assert.Error(t, err)
    })
    // ...