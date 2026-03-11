node {
    // Menarik kode dari GitHub (Fodly18/test-jenkins)
    checkout scm

    // 1. Build Stage
    stage("Build") {
        // Menggunakan composer terbaru yang mendukung PHP 8.4
        docker.image('composer:latest').inside('-u root') { 
            // Mengatasi error 'dubious ownership'
            sh 'git config --global --add safe.directory /var/lib/jenkins/workspace/laravel-dev'
            sh 'composer install --no-interaction --prefer-dist'
        }
    }

    stage("Testing") {
        // Menggunakan PHP 8.4 sesuai permintaan error tadi
        docker.image('php:8.4-cli').inside('-u root') {
            sh 'cp .env.example .env'
            sh 'php artisan key:generate'
            sh 'php artisan test'
        }
    }

    // 3. Deploy Stage
stage("Deploy") {
        docker.image('instrumentisto/rsync-ssh').inside('-u root') {
            sshagent (credentials: ['ssh-prod']) {
                // Kita ganti rsync agar mengabaikan pengecekan host key
                // Dan pastikan path rsync-nya benar
                sh "rsync -rav -e 'ssh -o StrictHostKeyChecking=no' --delete ./ ${env.USER_SERVER}@${env.PROD_HOST}:/home/${env.USER_SERVER}/laravel-jenkins/ --exclude=.env --exclude=storage --exclude=.git --exclude=vendor"
            }
        }
    }
}
