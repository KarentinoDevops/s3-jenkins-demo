pipeline {
  agent any

  environment {
    BUCKET     = 'karentino-jenkins-demo-london1'  // <-- change to yours
    AWS_REGION = 'eu-west-2'
    SITE_DIR   = 'dist'
  }

  options { timestamps() }

  stages {
    stage('Build') {
      steps {
        echo 'Building a tiny static site...'
        sh '''
          set -e
          rm -rf "$SITE_DIR"
          mkdir -p "$SITE_DIR"
          cat > "$SITE_DIR/index.html" << 'HTML'
          <!doctype html>
          <html lang="en">
          <head>
            <meta charset="utf-8" />
            <title>Hello, Karen!</title>
            <meta name="viewport" content="width=device-width, initial-scale=1" />
            <style>
              body { font-family: -apple-system, Arial, sans-serif; display:flex; align-items:center; justify-content:center; height:100vh; margin:0; }
              .card { text-align:center; padding:24px; border:1px solid #eee; border-radius:12px; box-shadow:0 6px 20px rgba(0,0,0,0.08); }
            </style>
          </head>
          <body>
            <div class="card">
              <h1>Hello, Karentino Abiona💛</h1>
              <p>Deployed by Jenkins (Build → Test → Deploy)</p>
            </div>
          </body>
          </html>
          HTML
        '''
        archiveArtifacts artifacts: "${SITE_DIR}/**", fingerprint: true
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -e
          test -f "$SITE_DIR/index.html"
          echo "index.html exists ✅"
        '''
      }
    }

    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'aws-creds',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh '''
            set -e
            export AWS_PAGER=""
            export AWS_DEFAULT_REGION="$(echo "$AWS_REGION" | xargs)"
            aws s3 sync "$SITE_DIR" "s3://$BUCKET" --delete

            REGION="$(aws s3api get-bucket-location --bucket "$BUCKET" --query 'LocationConstraint' --output text 2>/dev/null || true)"
            if [ "$REGION" = "None" ] || [ -z "$REGION" ]; then REGION="${AWS_DEFAULT_REGION:-eu-west-2}"; fi

            VHOST_URL="http://$BUCKET.s3-website-$REGION.amazonaws.com"
            echo "$VHOST_URL" > website_url.txt
            echo "Website URL: $VHOST_URL"
          '''
        }
      }
    }
  }

  post {
    success { script { echo "Open: ${readFile('website_url.txt').trim()}" } }
    failure { echo 'Something failed — check Console Output.' }
  }
}
